### Baseline Environment

At the start of the project, the application existed only as a local, frontend-only React Native / Expo application. The app was manually started in development mode with no backend services, no data persistence, and no reliability controls.

Any failure (process crash, machine shutdown, or network issue) would immediately take the application offline with no automated recovery.

**Evidence Artifacts**

- Local frontend running (screenshot)
- Development server terminal output
- Frontend-only project structure

Local Application Execution
Screenshots of the application running locally demonstrate that the frontend was functional but entirely dependent on a developer-managed environment. The application lifecycle was manually controlled, with no health checks, automated restarts, or external monitoring.

Development Server Process
Terminal output showing the Expo development server confirms that the application required manual startup and was operating in development mode. This highlights the absence of production-grade process management, fault detection, or recovery capabilities.

Frontend-Only Project Structure
The project file structure was captured to show that the repository contained only frontend code, with no backend services, APIs, containerization artifacts, or infrastructure configuration. This evidence establishes the architectural gap that the remainder of the project is designed to address.
