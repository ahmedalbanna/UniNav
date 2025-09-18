# UniNav
Interactive Campus Navigation System

|                    |                                                            |
| :----------------- | :--------------------------------------------------------- |
| **Document:**      | Project Blueprint                                          |
| **Version:**       | 1.0                                                        |
| **Project Leads:** | Ahmed Albanna (Frontend), Abdul Hafeez Al -Omari (Backend) |
| **Date:**          | September 16, 2025                                         |

---

## **1. Vision & Guiding Principles**

**Vision:** To create a fast, intuitive, and maintainable web-based navigation system for the university campus, eliminating the friction of wayfinding for students and visitors.

**Guiding Principles:**
*   **User-Centric Design:** The interface must be simple, responsive, and immediately understandable with zero training.
*   **Modularity:** Frontend and backend systems will be decoupled, communicating via a well-defined API. This allows for independent development and future scalability.
*   **Data-Driven:** All map points of interest and navigation paths will be stored in a central database, making the system easy to update and manage through an admin panel.
*   **Performance:** The application must load quickly and provide a smooth, lag-free experience when panning, zooming, and searching.

## **2. System Architecture**

The system follows a classic client-server model composed of three main parts: a **Frontend Web App**, a **Backend API**, and a **Database**. A fourth component, the **Admin Dashboard**, provides a UI for managing the backend.

```
+------------------------+        +------------------------+
|                        |        |                        |
|   User (Web Browser)   | <----->|  Frontend Web App      |
|                        |        |  (HTML, CSS, JS)       |
+------------------------+        +-----------+------------+
                                              |
                                              | (HTTP Requests via REST API)
                                              |
+------------------------+        +-----------v------------+
|                        |        |                        |
|   Administrator        | <----->|  Backend Server        |
|   (Web Browser)        |        |  (Laravel / PHP)       |
+------------------------+        +-----------+------------+
                                              |
                                              | (Database Queries)
                                              |
                                      +-------v--------+
                                      |                |
                                      |    MySQL       |
                                      |    Database    |
                                      +----------------+
```

## **3. Technology Stack**

*   **Frontend:**
    *   **Languages:** HTML5, CSS3, JavaScript (ES6+)
    *   **Mapping Library:** Leaflet.js (for map tiling, layers, and markers)
    *   **SVG Interaction:** Plain JavaScript or a lightweight library like D3.js for DOM manipulation of SVG elements (highlighting rooms, drawing paths).
    *   **Build Tool (Optional):** Vite or Webpack for bundling assets.

*   **Backend:**
    *   **Framework:** Laravel 11.x (PHP 8.2+)
    *   **Database:** MySQL 8.x
    *   **Server:** Nginx or Apache

*   **Development & Deployment:**
    *   **Version Control:** Git / GitHub
    *   **Package Managers:** Composer (PHP), NPM (JavaScript)
    *   **Hosting:** Shared Hosting with SSH access or a VPS (e.g., DigitalOcean).

## **4. Database Schema**

The database will be structured to logically store the campus hierarchy and navigation data.

**Table: `buildings`**
*   `id` (PK, INT, AUTO_INCREMENT)
*   `name` (VARCHAR) - e.g., "Faculty of Engineering"

**Table: `floors`**
*   `id` (PK, INT, AUTO_INCREMENT)
*   `building_id` (FK to `buildings.id`)
*   `name` (VARCHAR) - e.g., "Ground Floor"
*   `level` (INT) - e.g., 0, 1, 2 (for ordering)
*   `svg_path` (VARCHAR) - Path to the SVG file for this floor, e.g., "/maps/eng_floor_0.svg"

**Table: `locations` (Points of Interest)**
*   `id` (PK, INT, AUTO_INCREMENT)
*   `floor_id` (FK to `floors.id`)
*   `name` (VARCHAR) - e.g., "Room 101", "Dean's Office"
*   `svg_id` (VARCHAR) - **Critical Link.** The unique ID of the corresponding shape in the SVG file, e.g., "room-101".
*   `type` (ENUM) - 'classroom', 'lab', 'office', 'restroom', 'entrance'
*   `center_x` (FLOAT), `center_y` (FLOAT) - Coordinates for placing a map marker.

**Table: `nav_nodes` (Navigation Graph Nodes)**
*   `id` (PK, INT, AUTO_INCREMENT)
*   `floor_id` (FK to `floors.id`)
*   `x` (FLOAT), `y` (FLOAT) - Pixel coordinates on the SVG map.

**Table: `nav_edges` (Navigation Graph Connections)**
*   `id` (PK, INT, AUTO_INCREMENT)
*   `node_start_id` (FK to `nav_nodes.id`)
*   `node_end_id` (FK to `nav_nodes.id`)
*   `weight` (INT) - The calculated distance (cost) between the nodes.

## **5. API Endpoints (The Contract)**

The backend will expose the following RESTful API endpoints. All responses will be in JSON format.
**Base URL:** `/api/v1/`

*   **`GET /buildings`**
    *   **Description:** Fetches a list of all available buildings.
    *   **Response:** `[{ "id": 1, "name": "Faculty of Engineering" }, ...]`

*   **`GET /buildings/{id}/floors`**
    *   **Description:** Fetches all floors for a specific building.
    *   **Response:** `[{ "id": 1, "name": "Ground Floor", "level": 0, "svg_path": "/maps/eng_floor_0.svg" }, ...]`

*   **`GET /floors/{id}/locations`**
    *   **Description:** Fetches all named locations for a specific floor.
    *   **Response:** `[{ "id": 101, "name": "Room 101", "svg_id": "room-101", "type": "classroom" }, ...]`

*   **`GET /search?query={searchTerm}`**
    *   **Description:** Searches for locations across all floors and buildings.
    *   **Response:** `[{ "id": 101, "name": "Room 101", "floor_id": 1, "building_name": "Faculty of Engineering" }, ...]`

*   **`GET /floors/{id}/navigation`**
    *   **Description:** Fetches the navigation graph (nodes and edges) for a specific floor, needed for pathfinding on the frontend.
    *   **Response:** `{ "nodes": [...], "edges": [...] }`

## **6. Frontend Architecture & Components**

The frontend will be a Single Page Application (SPA) experience, structured around reusable components.

*   **`MapView.js`:** The main component responsible for loading and displaying the Leaflet map and the SVG floor plan overlay. It handles zooming, panning, and user clicks.
*   **`SearchBar.js`:** Handles user input, sends search requests to the API, and displays results. When a result is clicked, it communicates with `MapView` to highlight the location.
*   **`Controls.js`:** Contains the UI elements for selecting a building and switching between floors.
*   **`Pathfinding.js`:** A utility module containing the A* (or similar) pathfinding algorithm. It takes the navigation graph, a start, and an end point, and returns an array of coordinates for the path.
*   **`ApiService.js`:** A dedicated module for handling all communication with the backend API.
*   **`StateManagement.js`:** A simple global state object to keep track of the currently selected building, floor, and start/end points for navigation.

### **7. Backend Architecture (Laravel)**

*   **Models:** `Building`, `Floor`, `Location`, `NavNode`, `NavEdge` models will be created to correspond with the database tables.
*   **Controllers:** API controllers (e.g., `BuildingController`, `FloorController`) will handle incoming HTTP requests, fetch data from the models, and return JSON responses.
*   **Routing:** API routes will be defined in `routes/api.php` to map the endpoints to their respective controller methods.
*   **Admin Dashboard:** Built using Laravel's Blade templating engine and standard controllers. It will be protected by an authentication middleware. The dashboard will provide simple web forms for all CRUD operations on the database tables. A key feature will be an interface to visually add/remove `nav_nodes` on a floor map image.

## **8. Workflow & Task Breakdown**

*   **Sprint 0: Setup**
    *   Initialize GitHub repository with a `main` and `develop` branch.
    *   Set up local development environments (Laravel Valet/Herd, Node.js).
    *   Finalize database schema and API contract.
    *   Create initial vector traces (SVG) for one building's floors.

*   **Your Tasks (Frontend):**
    1.  Set up the basic HTML/CSS/JS project structure.
    2.  Implement the `MapView` to display a static SVG.
    3.  Develop the `Controls` to switch between different static SVGs.
    4.  Build the `ApiService` to fetch data from mock API endpoints.
    5.  Integrate with the live backend API.
    6.  Implement the search functionality.
    7.  Implement the pathfinding algorithm and path drawing on the map.

*   **Partner's Tasks (Backend):**
    1.  Set up the Laravel project and database migrations.
    2.  Build the Models and API Controllers.
    3.  Implement all API endpoints with dummy data.
    4.  Build the admin authentication system.
    5.  Create the CRUD interfaces in the Admin Dashboard for all tables.
    6.  Connect controllers to the database to serve real data.
    7.  Deploy the backend to the hosting environment.
