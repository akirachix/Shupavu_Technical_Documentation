 Glossary

Standardized terms, system variables, and vocabulary expressions used across the technical documentation of E-Madini.

## Platform & Domain Terms

| Term | Definition |
| :--- | :--- |
| **E-Madini** | The E-waste platform helping recyclers identify, assess, and recover critical materials from electronic waste. |
| **Producer** | A user role representing an organization (bank, school, hospital, office) that has e-waste to dispose of. |
| **Recycler** | A user role representing a company that collects and processes e-waste. |
| **Admin** | A user role that oversees the platform and system-level operations. |
| **Pickup Request** | A record connecting a producer's e-waste with a recycler for collection and processing. |
| **Device Model** | A registered electronic device record, including its category, brand, and material composition. |
| **Material Composition** | The breakdown of materials (metals, plastics, critical materials) estimated to be present in a device. |
| **Critical Materials** | Materials such as lithium, cobalt, nickel, copper, and rare-earth elements that are valuable for electromobility and energy technologies. |
| **Disposal/Processing Report** | A record documenting the outcome of a completed pickup and processing activity, used for compliance and traceability. |
| **NEMA Certificate** | An official disposal certificate generated after e-waste has been processed, referencing Kenya's National Environment Management Authority. |

## Backend Terms

| Term | Definition |
| :--- | :--- |
| **REST API** | An architectural style for exposing backend functionality over HTTP using resource-based endpoints. |
| **FastAPI** | The Python web framework used to build E-Madini's backend API. |
| **SQLAlchemy** | The ORM (Object-Relational Mapper) used to interact with the PostgreSQL database from Python. |
| **Pydantic Schema** | A data validation and serialization definition used to structure API requests and responses. |
| **Repository (pattern)** | A backend layer responsible for direct database operations, isolated from business logic. |
| **Service (layer)** | A backend layer that implements business logic, coordinating repositories and external integrations. |
| **JWT (JSON Web Token)** | A signed token used to authenticate users and authorize requests to protected endpoints. |
| **Alembic** | The migration tool used to version and apply changes to the database schema. |
| **LocationIQ** | The geocoding service used to convert addresses into latitude/longitude coordinates. |
| **YOLO (You Only Look Once)** | A computer vision model architecture used for detecting and classifying devices from images. |

## Frontend Terms

| Term | Definition |
| :--- | :--- |
| **Next.js App Router** | The routing system used by both web applications, where folders under `app/` map to URL routes. |
| **Client Component** | A React component that renders and runs in the browser, able to use state, effects, and browser APIs like `localStorage`. |
| **Middleware (`middleware.ts`)** | Next.js code that runs before a request completes, used here to help enforce route-level protections on the Dashboard. |
| **RequireAuth** | A client-side guard component on the Dashboard that checks for a valid stored token before rendering protected pages. |
| **CSS Modules** | A styling approach where CSS class names are automatically scoped to the component that imports them. |
| **Design Tokens** | Reusable CSS custom properties (e.g. `--color-primary`, `--radius-pill`) that define shared brand values across components. |
| **Hydration** | The process by which React attaches interactivity to server-rendered HTML in the browser; mismatches here can cause warnings or bugs. |
| **Fixture (testing)** | Predefined mock data (e.g. `mockPickupRequests`, `mockDeviceModels`) used to simulate backend responses in tests. |
| **Mock (testing)** | A stand-in for a real dependency (like an API route or auth token) used to isolate the behavior being tested. |

## Testing & QA Terms

| Term | Definition |
| :--- | :--- |
| **Playwright** | An end-to-end testing framework used to simulate real user interactions with the Dashboard in a browser. |
| **Cypress** | An end-to-end testing framework used to simulate real user interactions with the Informational Website in a browser. |
| **E2E (End-to-End) Test** | A test that verifies a complete user flow (e.g. signing in and reaching the dashboard) rather than an isolated unit of code. |
| **`test.describe`** | A Playwright construct used to group related test cases under a shared label. |
| **`describe` / `it`** | Cypress constructs used to group related test cases (`describe`) and define individual test cases (`it`) within a spec file. |
| **Test ID (e.g. `DASH-AUTH-001`)** | A short identifier used to uniquely reference a specific test case across documentation and test reports. |
| **Postman** | A tool used to manually test backend API endpoints outside of the automated test suite. |

## AI & Data Terms

| Term | Definition |
| :--- | :--- |
| **Computer Vision** | The AI technique used to identify a device type from an uploaded image. |
| **Confidence Score** | A value indicating how certain a model is about a prediction; used to decide whether to accept a result automatically or request confirmation. |
| **Random Forest** | A machine learning model used to estimate material composition from device features. |
| **Circular Economy** | An economic model focused on reusing and recovering materials rather than discarding them, central to E-Madini's mission. |
| **Electromobility** | Technologies related to electric transportation (e.g. EV batteries), which depend on critical materials recoverable from e-waste. |