# Ping Pong Multiplayer Game - Design Document

# Estimation

**Time Estimation**  
The project will be developed in the following phases:

| Phase                                        | Estimated Time |
|---------------------------------------------|----------------|
| Requirement Analysis                         | 2 hours        |
| System Design                                | 2 hours        |
| Environment Setup                            | 1 hour         |
| Development                                  | 2 days         |
| &nbsp;&nbsp;&nbsp;&nbsp;- Keycloak Integration (Login + Token Handling) | 2 hours        |
| &nbsp;&nbsp;&nbsp;&nbsp;- ASP.NET Web API Setup (Controllers, DI, etc.) | 2 hours        |
| &nbsp;&nbsp;&nbsp;&nbsp;- PostgreSQL Schema & EF Core Setup            | 2 hours        |
| &nbsp;&nbsp;&nbsp;&nbsp;- User Module (Auth Middleware)       | 1.5 hours      |
| &nbsp;&nbsp;&nbsp;&nbsp;- Game Score Module (Save + History)           | 2 hours        |
| &nbsp;&nbsp;&nbsp;&nbsp;- Leaderboard Endpoint                         | 1.5 hours      |
| &nbsp;&nbsp;&nbsp;&nbsp;- WPF Integration (Token-based API Calls)      | 2.5 hours      |
| Testing                                        | 2 hours        |
| Multi-Window Local Simulation (Multi-user)     | 1 hour         |
| Deployment & Debugging                         | 3 hours        |
| **Total Estimated Time**                       | **3 days**     |


---

## Design

This document outlines the design and implementation strategy for extending the existing WPF-based Ping Pong game into a multi-user system with centralized authentication, persistent data storage, and asynchronous gameplay. The new system leverages ASP.NET Web API, PostgreSQL, and Keycloak to replace the existing local CSV-based solution.

---

## Goals

- Migrate from local CSV file storage to a centralized PostgreSQL database.
- Implement secure user authentication using Keycloak.
- Replace file-based game history with a RESTful Web API connected to PostgreSQL.
- Enable multiple users to play the game simultaneously via separate windows.
- Add leaderboard support to display highest scores.
- Maintain asynchronous, responsive communication between WPF clients and the backend API.

---

## Current Implementation

- **User Authentication**: Uses `User.csv` file for validating usernames and passwords.
- **Game History**: Stored as `<username>.csv` in a `UserData` folder.
- **Single Player**: Only one user can play per instance.
- **WPF Application**: Handles gameplay, user interaction, and file I/O operations locally.

---

## Proposed Architecture

### Backend (ASP.NET Core Web API)
- **Authentication**: Delegated to Keycloak (OAuth 2.0 / OpenID Connect).
- ![image](https://github.com/user-attachments/assets/e02a7093-f65a-4291-98de-734643681b27)

- **Data Storage**: PostgreSQL database to store:
  - User accounts (linked with Keycloak).
  - Game history per user.
  - Global leaderboard with top scores.
- **API Endpoints**:
  - `POST /api/game/end`
  - `GET /api/user/history`
  - `GET /api/leaderboard`
  - `POST /api/user/register`
  - `POST /api/user/login`
### Frontend (WPF Game Client)
- Login via Keycloak.
- Communicates with Web API for:
  - ending games
  - Saving scores
  - Fetching user history and leaderboard
- Supports multiple windows for multi-user testing.

---
## Database design
![database_design](https://github.com/user-attachments/assets/59648401-dbba-47bf-89af-6ba5afbfa3c2)

## 🔄 How Data Is Processed

### 1. **User Authentication (via Keycloak)**
- WPF app shows a login screen.
- User enters username and password.
- Credentials are sent to **Keycloak**.
- On success, Keycloak returns:
  - **Access Token** (short-lived)
  - **Refresh Token** (used to get new access tokens)
- WPF app stores both tokens locally in memory.

---

### 2. **Calling the ASP.NET API**
- WPF sends API requests (e.g., `GET /api/leaderboard`) with the `Access Token` in the `Authorization` header:


  ```
  Authorization: Bearer <access_token>
  ```

  
- ASP.NET validates the token via **JWT middleware** using Keycloak’s public keys.
- The token contains a claim `sub`, which is the **Keycloak User ID**.

---

### 3. **Mapping Game Data to Users**
- The backend extracts the `sub` claim (`keycloak_id`) from the token.
- It looks up the user in the database using this `keycloak_id`.
- Game data (like scores, play history, etc.) is stored and associated with the `user.id` (which is either the same as `keycloak_id`, or linked to it).

---

### 4. **Saving Game History**
- After each game, the WPF client sends a request like `POST /api/games/save`.
- The payload includes:

  
  ```json
  {
    "score": 280,
    "duration": "00:02:30",
    "played_at": "2025-06-03T10:23:00Z"
  }
  ```
  
- The API links this to the authenticated user and saves it in the `game_scores` table.

---

### 5. **Leaderboard Generation**
- WPF app calls `GET /api/leaderboard`.
- The API runs a query (e.g., max score per user).
- Returns the top users with their scores.
- WPF displays this in a table view.

---

## Technologies Used

| Component       | Technology         |
|----------------|--------------------|
| Authentication | Keycloak           |
| Backend API    | ASP.NET Core       |
| Database       | PostgreSQL         |
| Frontend       | WPF (.NET)         |
| Communication  | HTTP (RESTful API) |

---

## Scope (To Be Completed in 2 Days)

### ✅ Planned & In Scope
1. **Keycloak Integration**  
   - Setup Keycloak server or realm.
   - Configure WPF client to authenticate users.

2. **Backend API**  
   - Create minimal ASP.NET Core Web API.
   - Define models for User, GameHistory, and Score.
   - Implement endpoints for:
     - Submitting game scores.
     - Fetching leaderboard.
     - Retrieving game history per user.

3. **Database**  
   - Create PostgreSQL schema with required tables.
   - Use EF Core to manage database access.

4. **WPF Client Integration**  
   - Replace local CSV usage with API calls.
   - Allow multiple windows for two different users.
   - Use asynchronous HTTP communication for responsiveness.

5. **Testing Setup**  
   - Launch two game windows simulating two players.
   - Validate async functionality and score submissions.

---

## Out of Scope (For Now)

The following features are considered out of scope for this initial implementation but may be added in the future:

- Real-time multiplayer over WebSockets.
- In-game chat or messaging.
- Admin panel for game management.
- Achievements or badges system.
- User profile customization.
- OAuth integration with external providers (e.g., Google login).
- Data analytics dashboard for gameplay trends.
- Cloud deployment and CI/CD pipelines.

---

## Conclusion

This document defines the short-term migration goals and the technical scope of the Ping Pong game modernization effort. The focus for the next two days is on setting up a basic client-server interaction with authentication and data persistence. This modular design ensures future scalability for advanced features.
