# Ping Pong Multiplayer Game - Design Document

## Overview

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
- Login via Keycloak (using embedded browser or external login).
- Communicates with Web API for:
  - Starting/ending games
  - Saving scores
  - Fetching user history and leaderboard
- Supports multiple windows for multi-user testing.

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
