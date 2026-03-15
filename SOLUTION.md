# Shiptivitas API – Solution Documentation

## Purpose
To help users manage their task log easier and increase daily active users by allowing freight managers to move cards between swimlanes and reorder them by priority.

---

## User Stories

- As a freight manager, I want to be able to move Cards around the board backwards or forwards in the process.
- As a freight manager, I want to be able to change the priority of Cards by arranging them from top to bottom on each column.

---

## Changes Made

### File: `server.js`

Updated the `PUT /api/v1/clients/:id` endpoint to handle:

1. **Moving a card from one swimlane to another** – updates `status` and reassigns priorities in both source and target lanes.
2. **Reordering a card within the same swimlane** – updates `priority` and shifts other cards accordingly.
3. **Persisting all changes to the SQLite database** – ensures position and order survive a page refresh.

---

## Acceptance Criteria

| # | Criteria | Status |
|---|----------|--------|
| 1 | Moving a card between swimlanes updates the database | ✅ Done |
| 2 | Reordering a card in the same swimlane updates the database | ✅ Done |
| 3 | Page refresh retains card position and order | ✅ Done |

---

## API Reference

### GET all clients
```
GET /api/v1/clients
GET /api/v1/clients?status=backlog
GET /api/v1/clients?status=in-progress
GET /api/v1/clients?status=complete
```

### GET single client
```
GET /api/v1/clients/:id
```

### UPDATE client status or priority
```
PUT /api/v1/clients/:id
Body: { "status": "in-progress", "priority": 1 }
```

---

## How to Test

### 1. Move card to a different swimlane
```bash
curl -X PUT http://localhost:3001/api/v1/clients/1 \
  -H "Content-Type: application/json" \
  -d '{"status": "complete", "priority": 1}'
```
✅ Expected: client 1 now appears in `complete` lane at priority 1

### 2. Reorder within the same lane
```bash
curl -X PUT http://localhost:3001/api/v1/clients/3 \
  -H "Content-Type: application/json" \
  -d '{"priority": 1}'
```
✅ Expected: client 3 moves to top of its lane, others shift down

### 3. Verify persistence after server restart
```bash
# Make a change
curl -X PUT http://localhost:3001/api/v1/clients/5 \
  -H "Content-Type: application/json" \
  -d '{"status": "backlog", "priority": 1}'

# Restart server (Ctrl+C then)
node server.js

# Verify data is still saved
curl http://localhost:3001/api/v1/clients/5
```
✅ Expected: client 5 remains in `backlog` at priority 1 after restart

---

## How to Run

```bash
# Install dependencies
npm install

# Start server
node server.js
# Server runs on http://localhost:3001
```

---

## Logic Summary

| Scenario | Behaviour |
|----------|-----------|
| Status only provided | Moves card to new lane, inserts at end |
| Priority only provided | Reorders within current lane |
| Both provided | Moves to new lane at exact priority position |
| Other cards in target lane | Priorities reassigned 1..N, no gaps or duplicates |
| Cards left in source lane | Priorities closed up after card is removed |
