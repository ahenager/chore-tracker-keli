# Chore Tracker API Documentation

This document outlines the REST API endpoints available in the Chore Tracker application.

---

## `GET /getUserList`

Retrieves a list of all users in the system, ordered alphabetically by name.

**Request Method:** `GET`

**Response:**
Returns an array of User objects.
```json
[
  {
    "Id": "string",
    "Name": "string"
  }
]
```

---

## `GET /retrieveTasks`

Retrieves the current day's tasks and the user's progress/goals. Also triggers a calculation date update for users if needed.

**Request Method:** `GET`

**Query Parameters:**
* `user` (string, required): The ID of the User.

**Response:**
Returns an object containing the user's available tasks for today and their goal progress.
```json
{
  "tasks": [
    {
      "Id": "string",
      "Name": "string",
      "Value": "number",
      "count": "number (number of times completed today)",
      "remaining": "number (percentage of time remaining for time-restricted tasks, or null)",
      "start": "string (formatted start time, or null)",
      "end": "string (formatted end time, or null)"
    }
  ],
  "goals": [
    {
      "daily": "number (Daily_Goal)",
      "weekly": "number (Weekly_Goal)",
      "day": "number (Daily_Total)",
      "week": "number (Weekly_Total)"
    }
  ]
}
```

---

## `POST /completeTask`

Marks a task (chore) as completed by a user, creating an `Assignment` record.

**Request Method:** `POST`

**Body Parameters (JSON):**
* `task` (string, required): The ID of the Chore to complete.
* `user` (string, required): The ID of the User completing the chore.

**Response:**
Returns the ID of the newly created Assignment and the user's updated goals/totals.
```json
{
  "id": "string (Assignment ID)",
  "goals": [
    {
      "daily": "number (Daily_Goal)",
      "weekly": "number (Weekly_Goal)",
      "day": "number (Daily_Total)",
      "week": "number (Weekly_Total)"
    }
  ]
}
```

---

## `POST /uncompleteTask`

Removes a previously completed task (deletes an `Assignment` record).

**Request Method:** `POST`

**Body Parameters (JSON):**
* `id` (string, required): The ID of the Assignment to remove.

**Response:**
Success (200 OK) with no content.

---

## `GET /getWeeklyData`

Retrieves aggregated chore data for a specific user and week, including weekly goal progress and a list of all assigned/completed tasks for that week.

**Request Method:** `GET`

**Query Parameters:**
* `user` (string, required): The ID of the User.
* `date` (string, required): A date string used to determine which week to query. The week is calculated as the ISO week containing this date (Monday through Sunday).

**Response:**
Returns the weekly goal, total points earned in the week, and an array of task objects (a FULL JOIN of available chores and completed assignments for the week).
```json
{
  "goal": "number (Weekly_Goal)",
  "total": "number (Total points from Assignments in the week)",
  "tasks": [
    {
      "id": "string (Chore ID)",
      "name": "string (Chore Name)",
      "count": "number (Number of times completed in the week)",
      "value": "number (Point value of the chore)"
    }
  ]
}
```
