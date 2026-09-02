# Backend Reference

The E-Madini backend provides the core services responsible for authentication, user management, device and material intelligence, database operations, pickup management, location processing, and disposal reporting.

The backend is designed as a modular REST API using **FastAPI**, with **PostgreSQL** as the primary database and SQLAlchemy providing database interaction. It acts as the central communication layer between the E-Madini web application, mobile application, AI services, and external integrations.

---

## Backend Overview

The backend is responsible for:

- User registration and authentication
- Role-based access control
- Recycler and producer account management
- Location and geocoding services
- E-waste and device registration
- AI-assisted device identification
- Material composition analysis
- Critical material recovery information
- Pickup request management
- Disposal and processing reports
- Database transactions
- API validation and error handling
- Integration with external services
- Secure communication between application components

The backend follows a layered architecture so that database operations, business logic, validation, and API routes remain separated.

---

## Technology Stack

| Layer | Technology |
| :--- | :--- |
| **Backend Framework** | FastAPI |
| **Programming Language** | Python 3.10+ |
| **Database** | PostgreSQL |
| **ORM** | SQLAlchemy |
| **Data Validation** | Pydantic |
| **Authentication** | JWT Bearer Authentication |
| **Password Security** | bcrypt |
| **Database Migrations** | Alembic |
| **Computer Vision** | Ultralytics YOLO |
| **Machine Learning** | XGBoost |
| **Geolocation** | LocationIQ |
| **API Documentation** | OpenAPI / Swagger UI |
| **Testing** | Cypress / Postman |
| **Deployment** | Heroku |
| **Version Control** | Git / GitHub |

---

# Backend Architecture

E-Madini uses a layered backend architecture.

The main request flow is:

```text
Client Application
       |
       v
    Router
       |
       v
    Schema
       |
       v
   Service
       |
       v
  Repository
       |
       v
   Database
```

External services and AI components can be triggered from the service layer:

```text
                    +----------------+
                    | Web Application|
                    +-------+--------+
                            |
                    +-------v--------+
                    |    FastAPI     |
                    |     Router     |
                    +-------+--------+
                            |
                    +-------v--------+
                    |    Schemas     |
                    |   Pydantic     |
                    +-------+--------+
                            |
                    +-------v--------+
                    |    Services    |
                    +---+--------+---+
                        |        |
              +---------+        +----------+
              |                              |
      +-------v-------+              +-------v-------+
      |  Repositories |              | AI / External |
      |               |              | Integrations  |
      +-------+-------+              +---------------+
              |
      +-------v-------+
      |   PostgreSQL  |
      +---------------+
```

This structure makes the backend easier to maintain, test, extend, and debug.

---

# Project Structure

The backend follows a modular structure where each major domain has its own directory.

```text
backend/
│
├── app/
│   │
│   ├── auth/
│   │   ├── models.py
│   │   ├── schemas.py
│   │   ├── repository.py
│   │   ├── service.py
│   │   └── router.py
│   │
│   ├── users/
│   │   ├── models.py
│   │   ├── schemas.py
│   │   ├── repository.py
│   │   ├── service.py
│   │   └── router.py
│   │
│   ├── locations/
│   │   ├── models.py
│   │   ├── schemas.py
│   │   ├── repository.py
│   │   ├── service.py
│   │   └── router.py
│   │
│   ├── pickup_requests/
│   │   ├── models.py
│   │   ├── schemas.py
│   │   ├── repository.py
│   │   ├── service.py
│   │   └── router.py
│   │
│   ├── device_models/
│   │   ├── models.py
│   │   ├── schemas.py
│   │   ├── repository.py
│   │   ├── service.py
│   │   └── router.py
│   │
│   ├── disposal_reports/
│   │   ├── models.py
│   │   ├── schemas.py
│   │   ├── repository.py
│   │   ├── service.py
│   │   └── router.py
│   │
│   ├── database.py
│   ├── dependencies.py
│   └── main.py
│
├── alembic/
│   ├── versions/
│   └── env.py
│
├── utilis/
│   ├── distance.py
│   ├── security.py
│   
│
├── .env
├── requirements.txt
└── README.md
```

---

# Layered Architecture

Each backend module separates responsibilities across several layers.

## 1. Models

Models define the database entities and their relationships using SQLAlchemy.

For example:

```python
class DeviceModel(Base):
    __tablename__ = "devicemodel"

    devicemodel_id = Column(String, primary_key=True)
    devicemodel_name = Column(String, nullable=False)
    device_type = Column(String, nullable=False)
    material_composition = Column(String)
    recycler_id = Column(String, nullable=False)
```

Models should contain database structure rather than application-specific business rules.

---

## 2. Schemas

Schemas define the structure of data entering and leaving the API.

E-Madini uses Pydantic schemas for request validation and response serialization.

Example:

```python
class DeviceModelResponse(BaseModel):
    devicemodel_id: str
    devicemodel_name: str
    device_type: str
    material_composition: str | None = None
    recycler_id: str
```

Schemas help ensure that API requests contain valid data before reaching the business logic.

---

## 3. Repositories

Repositories are responsible for database operations.

They provide an abstraction between application logic and SQLAlchemy queries.

Example:

```python
class DeviceModelRepository:

    def get(self, db: Session, devicemodel_id: str):
        statement = select(DeviceModel).where(
            DeviceModel.devicemodel_id == devicemodel_id
        )

        return db.scalar(statement)
```

Repositories should primarily handle:

- Creating records
- Reading records
- Updating records
- Deleting records where permitted
- Filtering records
- Database queries
- Transaction operations

---

## 4. Services

Services contain the application's business logic.

For example, when a recycler submits a device for analysis, the service may:

1. Receive the uploaded image.
2. Send the image to the computer vision model.
3. Identify the device.
4. Determine the likely device category.
5. Retrieve material information.
6. Calculate or retrieve critical material estimates.
7. Save the result.
8. Return the processed information.

Example:

```python
def process_device_scan(
    db: Session,
    file: UploadFile,
    recycler_id: str
):

    prediction = run_yolo_inference(file)

    composition = lookup_material_composition(
        prediction.device_type
    )

    return device_repository.create_device_model(
        db,
        prediction.name,
        prediction.device_type,
        composition,
        recycler_id
    )
```

The service layer is where E-Madini's core application behavior is implemented.

---

## 5. Routers

Routers expose functionality through HTTP endpoints.

Example:

```python
router = APIRouter(
    prefix="/device-models",
    tags=["Device Models"]
)


@router.post(
    "/scan",
    response_model=DeviceModelResponse,
    status_code=201
)
def scan_device(
    file: UploadFile,
    db: Session = Depends(get_db)
):

    return device_service.process_device_scan(
        db,
        file
    )
```

Routers should remain lightweight.

They should receive requests, validate input, call the appropriate service, and return the response.

---

# Database Connection

E-Madini uses PostgreSQL as its primary relational database.

The SQLAlchemy configuration is responsible for creating the database engine and managing database sessions.

A typical connection flow is:

```text
FastAPI
   |
   v
SQLAlchemy Session
   |
   v
PostgreSQL
```

Database connection details should never be hard-coded into application files.

They should be stored using environment variables.

Example:

```text
DATABASE_URL=postgresql://<username>:<password>@localhost:5432/emadini
```

---

# Authentication

E-Madini uses JWT-based authentication to protect authenticated resources.

Authentication follows the following process:

```text
User
 |
 | Email + Password
 v
POST /auth/login
 |
 v
Password Verification
 |
 v
JWT Generation
 |
 v
Access Token
 |
 v
Protected API Requests
```

The client includes the token in subsequent requests:

```text
Authorization: Bearer <token>
The token is delivered as an httpOnly cookie (`access_token`), not a header the browser sends it automatically on subsequent requests, and it's never exposed to client-side JavaScript.
```

---

## Password Security

Passwords must never be stored as plain text.

The backend uses password hashing before storing credentials.

Example:

```python
hashed_password = bcrypt.hash(password)
```

During authentication, the submitted password is compared against the stored hash.

```text
Submitted Password
        |
        v
Password Hash Verification
        |
   +----+----+
   |         |
 Valid     Invalid
   |         |
   v         v
Generate    Reject
JWT         Request

## Multi-Factor Authentication (OTP)

Login is not single-factor which a valid password only produces a temporary MFA token, not a session. The full flow:

1. `POST /auth/login` verifies the password, then generates a 6-digit OTP.
2. The OTP is encrypted (along with the user ID and a purpose flag) using a separate symmetric key, producing a short-lived `mfa_token`. This token self-expires after 1 minute.
3. The OTP is emailed to the user.
4. `POST /auth/verify-otp` checks the submitted OTP against the encrypted token using a timing-safe comparison, then issues the real access token.

Two protections apply during verification:

- Replay protection: once an OTP is successfully used, it can't be submitted again.
- Brute-force protection: after 5 failed attempts, the user must log in again to get a fresh OTP.

The resulting access token is valid for 180 minutes and is delivered as the httpOnly cookie described above.

```

---

# Role-Based Access Control

E-Madini supports different user roles.

Current roles include:

- `producer`
- `recycler`
- `admin`

Each role has different responsibilities and access permissions.

| Role | Primary Responsibility |
| :--- | :--- |
| **Producer** | Provides electronic equipment for recovery |
| **Recycler** | Processes e-waste and recovers valuable materials |
| **Admin** | Oversees the platform and manages system-level operations |

Authorization checks should be performed before allowing access to protected resources.

---

# Core Backend Modules

## Authentication Module

The authentication module manages:

- User login
- Password verification
- JWT generation
- Token validation
- Role identification
- Protected route access

Typical endpoints include:

```text
POST /auth/login
POST /auth/verify-otp
```

---

# User Management

The user module manages platform accounts.

Typical operations include:

```text
POST   /users/signup
GET    /users/{user_id}
PUT    /users/{user_id}
```

User records may include:

- User ID
- Email
- Password hash
- Role
- Organization name
- Phone number
- Location
- Account status
- Creation timestamp
- Update timestamp

---

# Location Services

E-Madini uses location information to associate organizations and operational activities with physical locations.

Location processing may use the LocationIQ geocoding service.

The flow is:

```text
Address
   |
   v
LocationIQ
   |
   v
Latitude + Longitude
   |
   v
PostgreSQL
```

Example location response:

```json
{
  "location_id": "LOC001",
  "latitude": -1.286389,
  "longitude": 36.817223,
  "display_name": "Nairobi, Kenya"
}
```

If an address cannot be resolved, the application should handle the error gracefully rather than terminating the request unexpectedly.

---

# Device and Material Intelligence

The device module is one of the core components of E-Madini.

It supports recyclers by transforming device information into useful material intelligence.

The processing pipeline is:

```text
Device Image
     |
     v
Computer Vision Model
     |
     v
Device Identification
     |
     v
Device Type
     |
     v
Material Composition
     |
     v
Critical Material Information
     |
     v
Recycler Decision Support
```

The purpose is not simply to identify electronic devices.

The system uses device information to help recyclers understand the potential recoverable materials contained within e-waste.

---

# Computer Vision Integration

E-Madini can use an Ultralytics YOLO model for image-based device identification.

The model receives an uploaded image and returns detection information.

Example conceptual response:

```json
{
  "device_type": "smartphone",
  "confidence": 0.91
}
```

The backend can then use the detected device type to retrieve additional material information.

---

## Confidence Handling

Computer vision predictions should not automatically be treated as correct.

The backend can use a confidence threshold to identify uncertain predictions.

For example:

```text
Confidence >= threshold
        |
        v
Accept prediction

Confidence < threshold
        |
        v
Request recycler confirmation
```

This provides a safeguard against incorrect automated classification.

---

# Material Intelligence

Once a device has been identified, E-Madini can associate it with material composition information.

Material information may include:

- Material category
- Estimated recoverable materials
- Critical materials
- Material recovery potential
- Device condition
- Estimated recovery rate
- Potential material value
- Recycling score

This information is intended to help recyclers make better processing and recovery decisions.

---

# Critical Materials for Electromobility

A major purpose of E-Madini is to improve the recovery of materials that can contribute to future industrial applications.

These may include materials associated with:

- Batteries
- Electronic circuits
- Motors
- Magnets
- Energy storage systems
- Electronic components

Examples of critical materials that may be relevant include:

- Lithium
- Cobalt
- Nickel
- Copper
- Rare-earth elements

The platform provides a digital way of connecting electronic waste information with material recovery opportunities.

---

# Pickup Request Management

Pickup requests connect producers and recycling companies.

The general process is:

```text
Producer
   |
   | Pickup Request
   v
E-Madini Backend
   |
   v
Recycler
   |
   | Collection / Processing
   v
E-Madini
   |
   v
Processing Record
```

A pickup request may contain:

- Request ID
- Producer ID
- Recycler ID
- Scheduled date
- Location
- Quantity
- E-waste image
- Request status
- Creation timestamp
- Completion timestamp

---

The `status` field drives the request lifecycle and only moves in one direction:

```text
PENDING → ACCEPTED → COMPLETED
   │          │
   ▼          ▼
REJECTED   REJECTED
```

A request starts as `PENDING`. The assigned recycler can accept (`ACCEPTED`) or reject (`REJECTED`) it. An accepted request can only move to `COMPLETED` or `REJECTED` it can never go back to `PENDING`.

---

# Disposal and Processing Reports

The disposal report module records information associated with completed processing activities.

Reports can be linked to their originating requests.

The relationship is:

```text
Pickup Request
      |
      v
Processing Activity
      |
      v
Disposal / Processing Report
```

A report may contain:

- Report ID
- Request ID
- Issue date
- Report file URL
- Processing information

These records support traceability and accountability within the recycling workflow.

---

# API Design

E-Madini follows REST API principles.

Endpoints are grouped according to resources.

Example resource groups:

```text
/auth
/users
/locations
/pickup-requests
/device-models
/disposal-reports
```

HTTP methods are used according to the operation being performed.

| Method | Purpose |
| :--- | :--- |
| `GET` | Retrieve information |
| `POST` | Create a resource |
| `PUT` | Update a resource |
| `PATCH` | Partially update a resource |
| `DELETE` | Remove a resource where permitted |

---

# API Documentation

FastAPI automatically generates OpenAPI documentation.

When running the backend locally, Swagger UI is normally available at:

```text
http://127.0.0.1:8000/docs
```

The ReDoc interface is normally available at:

```text
http://127.0.0.1:8000/redoc
```

These interfaces allow developers to inspect available endpoints, request parameters, response structures, and authentication requirements.

---

# Example API Request

A device scanning request may use a multipart form-data request.

Example:

```text
POST /device-models/scan
```

Request:

```text
Authorization: Bearer <token>

Content-Type: multipart/form-data

file: device-image.jpg
```

A successful response could look like:

```json
{
  "devicemodel_id": "DEV001",
  "devicemodel_name": "Example Device",
  "device_type": "smartphone",
  "material_composition": "battery, copper, aluminium",
  "recycler_id": "REC001"
}
```

---

# Error Handling

The backend uses HTTP status codes to communicate request results.

| Status | Meaning | Typical Cause |
| :--- | :--- | :--- |
| **200** | Success | Request completed successfully |
| **201** | Created | New resource successfully created |
| **400** | Bad Request | Invalid request structure |
| **401** | Unauthorized | Missing or invalid authentication |
| **403** | Forbidden | User does not have permission |
| **404** | Not Found | Requested resource does not exist |
| **422** | Validation Error | Request failed Pydantic validation |
| **500** | Internal Server Error | Unexpected backend failure |

Errors should return meaningful messages while avoiding exposure of sensitive implementation details.

---

# Environment Variables

Sensitive configuration values must not be committed to GitHub.

Create a local `.env` file for development.

Example:

```text
DATABASE_URL=postgresql://<user>:<password>@localhost:5432/emadini

JWT_SECRET_KEY=<your-secret-key>

JWT_ALGORITHM=HS256

LOCATIONIQ_API_KEY=<your-locationiq-key>
```

The repository should contain an `.env.example` file instead of the real secrets.

Example:

```text
DATABASE_URL=
JWT_SECRET_KEY=
JWT_ALGORITHM=HS256
LOCATIONIQ_API_KEY=
```

Never commit:

```text
.env
```

to the repository.

---

# Requirements Installation

Install backend dependencies using:

```bash
pip install -r requirements.txt
```

If a virtual environment is being used:

```bash
python -m venv venv
```

Activate it on Linux/macOS:

```bash
source venv/bin/activate
```

Then install dependencies:

```bash
pip install -r requirements.txt
```

---

# Database Migrations

E-Madini uses Alembic to manage database schema changes.

To apply existing migrations:

```bash
alembic upgrade head
```

To create a new migration after changing database models:

```bash
alembic revision --autogenerate -m "describe the change"
```

Then apply the migration:

```bash
alembic upgrade head
```

Migration files should be committed to version control so that development and production databases can remain synchronized.

---

# Running the Backend Locally

From the backend project directory:

```bash
uvicorn app.main:app --reload
```

The backend should then be accessible at:

```text
http://127.0.0.1:8000
```

Swagger API documentation:

```text
http://127.0.0.1:8000/docs
```

The `--reload` option automatically restarts the development server when source files change.

It should primarily be used during development.

---

# Backend Startup Checklist

Before starting the backend, verify:

```text
[ ] Python environment is activated
[ ] Dependencies are installed
[ ] PostgreSQL is running
[ ] Database exists
[ ] .env file is configured
[ ] Database URL is correct
[ ] API keys are available
[ ] Database migrations are applied
```

Then start the application:

```bash
uvicorn app.main:app --reload
```

---

# Testing the Backend

Backend testing should cover both individual components and complete API workflows.

Recommended testing areas include:

- Authentication
- User registration
- Role authorization
- Database operations
- Location processing
- Pickup requests
- Device identification
- Material intelligence
- Disposal reports
- Error handling

---

## Automated Tests

Pytest can be used to execute automated backend tests.

Example:

```bash
pytest
```

For coverage:

```bash
pytest --cov=app
```

A successful test suite should verify that the backend behaves correctly before deployment.

---

# API Testing with Postman

Postman can be used to manually validate API endpoints.

Typical testing workflow:

```text
1. Start backend
2. Open Postman
3. Configure base URL
4. Create/login user
5. Store authentication token
6. Send authenticated requests
7. Verify response status
8. Verify response body
```

Example environment variables:

```text
base_url
producer_token
recycler_token
admin_token
producer_id
recycler_id
location_id
pickup_request_id
device_model_id
report_id
```

This makes it easier to test different user roles without repeatedly entering values manually.

---

# Git Branching Strategy

Backend development should use feature-based branches.

Examples:

```text
feature/device-scanning
feature/material-intelligence
feature/recycler-dashboard-api
bugfix/authentication-error
bugfix/location-api
```

The general workflow is:

```text
main
 |
 +---- feature/new-feature
 |
 |     Development
 |
 |     Testing
 |
 +---- Pull Request
 |
 +---- Review
 |
 +---- Merge
```

Developers should avoid committing directly to `main` unless the project workflow explicitly permits it.

---

# Pull Request Checklist

Before opening a pull request:

```text
[ ] Feature is complete
[ ] Code follows project structure
[ ] Tests have been added or updated
[ ] Existing tests pass
[ ] No secrets are committed
[ ] Environment variables are documented
[ ] Database migrations are included if required
[ ] API documentation is updated if required
[ ] Code has been reviewed locally
```

---

# Deployment

The backend can be deployed as a production FastAPI service.

The production environment should provide:

- Python runtime
- PostgreSQL database
- Environment variables
- API credentials
- Production server configuration
- HTTPS
- Database migrations

---

## Production Server

For production, the application should not rely on the development reload server.

A production deployment can use a process manager such as Gunicorn with Uvicorn workers.

Example:

```bash
gunicorn app.main:app \
    -k uvicorn.workers.UvicornWorker
```

The exact command may vary depending on the deployment platform.

---

# Heroku Deployment

The backend can be deployed to Heroku.

A typical deployment workflow is:

```text
Local Development
       |
       v
Git Commit
       |
       v
GitHub
       |
       v
Heroku
       |
       v
Production FastAPI API
       |
       v
PostgreSQL
```

Before deployment, configure production environment variables in the hosting platform.

Do not upload the local `.env` file.

---

## Heroku Configuration

Production configuration should include values such as:

```text
DATABASE_URL
JWT_SECRET_KEY
JWT_ALGORITHM
LOCATIONIQ_API_KEY
```

These should be configured using the hosting provider's environment/configuration settings.

---

# Production Deployment Checklist

Before deploying:

```text
[ ] All tests pass
[ ] Production database is available
[ ] Environment variables are configured
[ ] Secrets are not committed
[ ] Database migrations are ready
[ ] API configuration is correct
[ ] Debug/reload settings are disabled
[ ] Production server command is configured
[ ] Swagger/OpenAPI endpoints are verified
[ ] Frontend and mobile applications point to the production API
```

---

# Deployment Process

The recommended deployment sequence is:

```text
1. Develop feature
        |
        v
2. Run local tests
        |
        v
3. Test API with Postman
        |
        v
4. Commit changes
        |
        v
5. Push to GitHub
        |
        v
6. Review Pull Request
        |
        v
7. Merge approved changes
        |
        v
8. Deploy backend
        |
        v
9. Run database migrations
        |
        v
10. Verify production API
```

---

# Production Verification

After deployment, verify:

```text
[ ] API is reachable
[ ] Authentication works
[ ] Database connection works
[ ] Recycler endpoints work
[ ] Device scanning works
[ ] Material intelligence works
[ ] Location services work
[ ] Reports can be generated
[ ] Frontend can communicate with backend
[ ] Mobile application can communicate with backend
```

---

# Security Practices

Backend security is critical because the API handles authentication information, organizational data, operational records, and material intelligence.

The following practices should be maintained:

- Never store plaintext passwords.
- Never commit API keys.
- Never commit JWT secrets.
- Use HTTPS in production.
- Validate all incoming data.
- Authenticate protected endpoints.
- Authorize requests according to user roles.
- Avoid exposing internal exception details.
- Keep dependencies updated.
- Restrict database access.
- Use secure environment variables.
- Validate uploaded files.
- Apply appropriate file-size limits.
- Maintain useful application logs without exposing sensitive information.

---

# API Design Principles

The E-Madini backend should follow these principles:

### Separation of Concerns

Each component should have one primary responsibility.

### Reusability

Common functionality should be implemented once and reused.

### Validation

User input should be validated before being processed.

### Traceability

Important operations should produce records that can be traced back to their originating user or transaction.

### Security

Authentication and authorization should be applied to protected resources.

### Maintainability

New functionality should fit into the existing modular architecture rather than creating unnecessary dependencies between modules.

---

# Backend Development Workflow

A typical development cycle is:

```text
Understand Requirement
        |
        v
Design API
        |
        v
Create / Update Model
        |
        v
Create Schema
        |
        v
Implement Repository
        |
        v
Implement Service
        |
        v
Create Router
        |
        v
Write Tests
        |
        v
Test API
        |
        v
Review
        |
        v
Deploy
```

---

# Integration Points

The backend acts as the integration layer between E-Madini components.

```text
                    +------------------+
                    | E-Madini Web App |
                    +--------+---------+
                             |
                             |
                    +--------v---------+
                    |     FastAPI      |
                    |     Backend      |
                    +--+----+----+-----+
                       |    |    |
              +--------+    |    +---------+
              |             |              |
       +------v------+ +----v-----+ +------v------+
       | PostgreSQL  | | AI/ML    | | LocationIQ  |
       |             | | Services | |             |
       +-------------+ +----------+ +-------------+
                             |
                      +------v------+
                      | E-Madini    |
                      | Material    |
                      | Intelligence|
                      +-------------+
```

---

# External Integration

## LocationIQ Integration

E-Madini never stores raw coordinates entered by hand every `Location` row is created by geocoding a plain-text address through [LocationIQ](https://locationiq.com/). This happens in exactly two places: the `POST /locations/` endpoint, and internally whenever a pickup request is created from an address (`create_location_from_address()` is shared by both).

### Where it lives

All of the integration logic is in one place `emadini/services/location_service.py` routers never call LocationIQ directly.

```python
LOCATIONIQ_KEY = os.getenv("LOCATIONIQ_KEY")
BASE_URL = os.getenv("BASE_URL")  

class LocationService:
    def __init__(self, db: Session):
        self.db = db
        self.repo = LocationRepository(db)

    def create_location(self, data: LocationCreate) -> Location:
        return self.create_location_from_address(data.address)

    def create_location_from_address(self, address: str) -> Location:
        """Geocode an address via LocationIQ and persist the result.
        Reused by both the /locations endpoint and pickup-request creation."""
        ...
```

### Request flow

```text
address: str
      │
      ▼
GET {BASE_URL}?key={LOCATIONIQ_KEY}&q={address}&format=json&limit=1
      │
      ▼
LocationIQ returns a JSON array; first result is used
      │
      ▼
lat, lon, display_name extracted
      │
      ▼
Location row created:  Location(location_id, address, latitude, longitude, display_name)
```

The actual outbound call, with a hard 10-second timeout so a slow geocoder can't hang a request indefinitely:

```python
params = {"key": LOCATIONIQ_KEY, "q": address, "format": "json", "limit": 1}

try:
    response = requests.get(BASE_URL, params=params, timeout=10)
except requests.RequestException:
    raise HTTPException(status_code=503, detail="Geocoding provider is temporarily unreachable.")
```

### Every failure mode is handled explicitly

This isn't a bare `try/except` each LocationIQ response is mapped to a specific, meaningful API error:

| LocationIQ response | E-Madini raises | Meaning |
|---|---|---|
| Network error / timeout | `503 Service Unavailable` | LocationIQ is unreachable |
| `401` from LocationIQ | `401 Unauthorized` | `LOCATIONIQ_KEY` is invalid, expired, or over quota |
| `404` from LocationIQ | `400 Bad Request` | The address genuinely couldn't be geocoded |
| `429` from LocationIQ | `429 Too Many Requests` | LocationIQ's own rate limit was hit, passed straight through to the caller |
| Any other `4xx/5xx` | `400 Bad Request` | Generic "address validation failed (status N)" |
| `200` but empty/malformed body | `400 Bad Request` | Response didn't contain a usable `lat`/`lon`/`display_name` |
| `LOCATIONIQ_KEY` or `BASE_URL` not set | `500 Internal Server Error` | Server misconfiguration, checked before the request is even made |

```python
if not LOCATIONIQ_KEY or not BASE_URL:
    raise HTTPException(status_code=500, detail="Server misconfiguration: LOCATIONIQ_KEY/BASE_URL is not set.")
```

### Why this matters beyond `/locations`

Because `create_location_from_address()` is reused by `PickupRequestService.create_request()` (see [Service layer](#service-layer)), **every pickup request creation also depends on LocationIQ being reachable and correctly configured**. A LocationIQ outage doesn't just break `POST /locations/` it also blocks producers from submitting new pickup requests, since the request's `Location` and its nearest-recycler assignment both depend on a successful geocode first.

### Required configuration

| Variable | Purpose |
|---|---|
| `LOCATIONIQ_KEY` | Your LocationIQ API key |
| `BASE_URL` | LocationIQ's geocoding/search endpoint (e.g. `https://us1.locationiq.com/v1/search.php`) |

Both are validated at request time (not startup) and an unset key surfaces as a `500` on the first call that needs geocoding, rather than preventing the app from booting.


---

## Computer Vision

The backend can communicate with an Ultralytics YOLO model to process images submitted by recyclers.

Purpose:

- Device identification
- Object detection
- Device classification
- Confidence scoring

---

## Material Intelligence

The backend can combine device information with material datasets and machine learning models to estimate material recovery potential.

Potential processing flow:

```text
Device Information
       |
       v
Feature Preparation
       |
       v
ML / Material Model
       |
       v
Material Prediction
       |
       v
Recovery Information
       |
       v
Recycler Dashboard
```

---

# API Response Principles

API responses should be:

- Predictable
- Structured
- Validated
- Consistent
- Easy for frontend and mobile clients to consume

Example:

```json
{
  "success": true,
  "data": {
    "device_type": "laptop",
    "confidence": 0.94
  },
  "message": "Device processed successfully"
}
```

The exact response structure should remain consistent across related endpoints.

---

# Logging and Monitoring

The backend should record useful operational information without exposing sensitive credentials.

Useful events include:

- Application startup
- Authentication failures
- API errors
- Database errors
- AI processing failures
- External API failures
- Important processing operations

Sensitive values such as passwords, JWT secrets, and API keys must never be written to logs.

---

# Common Development Issues

## Database Connection Failure

Check:

```text
DATABASE_URL
PostgreSQL status
Database name
Username
Password
Port
```

---

## Migration Failure

Check that:

```bash
alembic upgrade head
```

is being executed against the intended database.

Also verify that model changes have corresponding migration files.

---

## 422 Validation Error

A `422` response usually indicates that the request does not match the Pydantic schema.

Check:

- Field names
- Required fields
- Data types
- Request format
- Multipart form fields
- JSON structure

---

## 401 Unauthorized

Check:

```text
Authorization: Bearer <token>
```

Also verify that:

- The token exists.
- The token has not expired.
- The token is correctly signed.
- The authentication dependency is being applied correctly.

---

## 403 Forbidden

The user may be authenticated but does not have permission to access the requested resource.

Verify the user's role.

---

## 500 Internal Server Error

Check backend logs for the underlying exception.

Common causes include:

- Database failures
- Missing environment variables
- Incorrect external API configuration
- Unexpected application exceptions
- Invalid AI model configuration

---

# Backend Quality Standards

Every backend contribution should aim for:

- Clear naming
- Small and focused functions
- Separation between API and business logic
- Strong request validation
- Meaningful error handling
- Automated tests
- Secure configuration
- Consistent API responses
- Clear documentation

---

# Summary

The E-Madini backend is the central service layer connecting the platform's applications, database, AI capabilities, and external services.

Its main responsibilities are to:

1. Authenticate and authorize users.
2. Manage platform data.
3. Connect recyclers with operational information.
4. Process device information.
5. Support material intelligence.
6. Help identify potential critical material recovery opportunities.
7. Maintain traceable records.
8. Expose reliable REST APIs.
9. Protect sensitive information.
10. Provide a foundation for scalable e-waste recovery and material intelligence.

The backend should remain modular so that additional recycling intelligence, material datasets, AI models, and platform capabilities can be added without requiring a complete redesign of the system.