"use strict";

const API_BASE = "http://localhost:8080";

const locations = [
    {id:1, name:"Security Office", x:390, y:510},
    {id:2, name:"Parking", x:390, y:445},
    {id:3, name:"Administrative Block", x:465, y:410},
    {id:4, name:"Library", x:330, y:405},
    {id:5, name:"Learning Centre 2", x:245, y:355},
    {id:6, name:"Halls of Residence", x:620, y:105},
    {id:7, name:"Sports Grounds", x:390, y:270},
    {id:8, name:"Learning Centre 1", x:490, y:230},
    {id:9, name:"Temple", x:390, y:110},
    {id:10, name:"Play Courts", x:670, y:330},
    {id:11, name:"Food Court", x:555, y:290},
    {id:12, name:"University Parking", x:760, y:130},
    {id:13, name:"Student Activity Centre", x:525, y:330}
];

const roads = [
    [1,2,65],
    [2,3,95],
    [2,4,75],
    [4,5,110],
    [3,7,145],
    [2,7,175],
    [7,8,120],
    [8,9,155],
    [8,6,170],
    [8,11,95],
    [11,10,115],
    [10,13,55],
    [13,3,105],
    [10,6,120],
    [6,12,135],
    [11,12,210],
    [7,9,190],
    [7,13,150]
];

const $ = id => document.getElementById(id);
const NS = "http://www.w3.org/2000/svg";

const startSelect = $("start");
const destinationSelect = $("destination");
const algorithmSelect = $("algorithm");

const roadsLayer = $("roads");
const locationsLayer = $("locations");

function svgElement(tag, attributes = {}) {
    const element = document.createElementNS(NS, tag);

    for (const [key, value] of Object.entries(attributes)) {
        element.setAttribute(key, value);
    }

    return element;
}

function setStatus(message, type = "") {
    $("status").textContent = message;
    $("status").className = ("status " + type).trim();
}

function populateDropdowns() {
    [startSelect, destinationSelect].forEach(select => {
        select.replaceChildren();

        locations.forEach(location => {
            const option = document.createElement("option");

            option.value = location.id;
            option.textContent =
                `${location.id}. ${location.name}`;

            select.appendChild(option);
        });
    });

    startSelect.value = "1";
    destinationSelect.value = "8";
}

function drawRoads() {
    roadsLayer.replaceChildren();

    roads.forEach(([a, b, distance]) => {
        const first = locations.find(location => location.id === a);
        const second = locations.find(location => location.id === b);

        const group = svgElement("g", {
            class: "road-group",
            "data-first": a,
            "data-second": b
        });

        group.appendChild(svgElement("line", {
            x1: first.x,
            y1: first.y,
            x2: second.x,
            y2: second.y,
            class: "road-edge"
        }));

        group.appendChild(svgElement("line", {
            x1: first.x,
            y1: first.y,
            x2: second.x,
            y2: second.y,
            class: "road"
        }));

        const label = svgElement("text", {
            x: (first.x + second.x) / 2,
            y: (first.y + second.y) / 2 - 7,
            class: "road-label",
            "text-anchor": "middle"
        });

        label.textContent = `${distance} m`;

        group.appendChild(label);
        roadsLayer.appendChild(group);
    });
}

function clearRoute() {
    document.querySelectorAll(".road").forEach(road => {
        road.classList.remove("route-road");
    });

    document.querySelectorAll(".road-label").forEach(label => {
        label.classList.remove("route-distance");
    });

    document.querySelectorAll(".location-node").forEach(node => {
        node.classList.remove(
            "start-location",
            "destination-location",
            "route-location"
        );
    });

    $("routeResults").hidden = true;
}

function markSelectedLocations(path = []) {
    const routeIds = path.map(item =>
        Number(typeof item === "object" ? item.id : item)
    );

    document.querySelectorAll(".location-node").forEach(node => {
        const id = Number(node.dataset.location);

        if (routeIds.includes(id)) {
            node.classList.add("route-location");
        }

        if (id === Number(startSelect.value)) {
            node.classList.add("start-location");
        }

        if (id === Number(destinationSelect.value)) {
            node.classList.add("destination-location");
        }
    });
}

function drawLocations() {
    locationsLayer.replaceChildren();

    locations.forEach(location => {
        const node = svgElement("g", {
            class: "location-node",
            "data-location": location.id,
            tabindex: "0",
            role: "button",
            "aria-label": location.name
        });

        node.appendChild(svgElement("circle", {
            cx: location.x,
            cy: location.y,
            r: 15,
            class: "node-halo"
        }));

        node.appendChild(svgElement("circle", {
            cx: location.x,
            cy: location.y,
            r: 10,
            class: "node-core"
        }));

        const number = svgElement("text", {
            x: location.x,
            y: location.y,
            class: "node-id"
        });

        number.textContent = location.id;
        node.appendChild(number);

        const label = svgElement("text", {
            x: location.x + 18,
            y: location.y - 16,
            class: "node-name"
        });

        if (location.x > 720) {
            label.setAttribute("x", location.x - 18);
            label.setAttribute("text-anchor", "end");
        }

        label.textContent = location.name;
        node.appendChild(label);

        const selectDestination = () => {
            if (Number(startSelect.value) === location.id) {
                setStatus(
                    "This is already your starting point. Choose a different destination."
                );
                return;
            }

            destinationSelect.value = String(location.id);

            clearRoute();
            markSelectedLocations();

            setStatus(
                `${location.name} selected. Click Find Route.`
            );
        };

        node.addEventListener("click", selectDestination);

        node.addEventListener("keydown", event => {
            if (event.key === "Enter" || event.key === " ") {
                event.preventDefault();
                selectDestination();
            }
        });

        locationsLayer.appendChild(node);
    });
}

function highlightRoute(path) {
    clearRoute();

    for (let i = 0; i < path.length - 1; i++) {
        const a = Number(path[i].id);
        const b = Number(path[i + 1].id);

        document.querySelectorAll(".road-group").forEach(group => {
            const first = Number(group.dataset.first);
            const second = Number(group.dataset.second);

            const matches =
                (first === a && second === b) ||
                (first === b && second === a);

            if (matches) {
                group.querySelector(".road")
                    .classList.add("route-road");

                group.querySelector(".road-label")
                    .classList.add("route-distance");
            }
        });
    }

    markSelectedLocations(path);
}

function displayResults(data) {
    const names = {
        dijkstra: "Dijkstra — shortest weighted distance",
        bfs: "BFS — fewest road segments",
        dfs: "DFS — depth-first traversal"
    };

    $("routeTitle").textContent =
        `${data.path[0].name} → ${data.path[data.path.length - 1].name}`;

    const unit = data.distanceUnit === "edges"
        ? (data.distance === 1 ? "edge" : "edges")
        : "m";

    $("routeDistance").textContent =
        `${data.distance} ${unit}`;

    $("algorithmTag").textContent =
        names[data.algorithm] || data.algorithm;

    const list = $("directions");
    list.replaceChildren();

    data.path.forEach(location => {
        const item = document.createElement("li");
        item.textContent = location.name;
        list.appendChild(item);
    });

    $("routeResults").hidden = false;
}

async function calculateRoute(event) {
    if (event) {
        event.preventDefault();
    }

    const start = Number(startSelect.value);
    const destination = Number(destinationSelect.value);
    const algorithm = algorithmSelect.value;

    if (start === destination) {
        clearRoute();
        markSelectedLocations();

        setStatus(
            "Starting point and destination must be different.",
            "error"
        );

        return;
    }

    setStatus("Calculating route...");
    $("mapStatus").textContent = "Calculating route...";

    try {
        const url =
            `${API_BASE}/route?start=${start}` +
            `&destination=${destination}` +
            `&algorithm=${encodeURIComponent(algorithm)}`;

        const response = await fetch(url);
        const data = await response.json();

        if (!response.ok || data.error) {
            throw new Error(
                data.error || `Server error: ${response.status}`
            );
        }

        if (!Array.isArray(data.path) || data.path.length < 2) {
            throw new Error("No valid route was returned.");
        }

        highlightRoute(data.path);
        displayResults(data);

        setStatus(
            "Route found! The green line shows your selected path.",
            "success"
        );

        $("mapStatus").textContent =
            `${data.path.length} locations in the selected route`;

    } catch (error) {
        console.error("Route calculation failed:", error);

        clearRoute();
        markSelectedLocations();

        setStatus(
            "Cannot connect to the C++ server. Start campus.exe and keep its terminal open. " +
            error.message,
            "error"
        );

        $("mapStatus").textContent = "Backend unavailable";
    }
}

async function checkServer() {
    try {
        const response = await fetch(`${API_BASE}/health`);

        if (!response.ok) {
            throw new Error("Health check failed");
        }

        const data = await response.json();

        if (!data.ok) {
            throw new Error("Unexpected server response");
        }

        $("connectionDot").className = "online";
        $("connectionText").textContent = "Backend connected";

        setStatus(
            "Connected to C++ server. Select locations to find a route.",
            "success"
        );

        $("mapStatus").textContent = "Map ready";

    } catch (error) {
        $("connectionDot").className = "offline";
        $("connectionText").textContent = "Backend offline";

        setStatus(
            "C++ server is not running. Start campus.exe in the MSYS2 UCRT64 terminal, then refresh this website.",
            "error"
        );

        $("mapStatus").textContent = "Waiting for backend";
    }
}

function swapLocations() {
    const oldStart = startSelect.value;

    startSelect.value = destinationSelect.value;
    destinationSelect.value = oldStart;

    clearRoute();
    markSelectedLocations();

    setStatus("Locations swapped. Click Find Route.");
}

function resetMap() {
    startSelect.value = "1";
    destinationSelect.value = "8";
    algorithmSelect.value = "dijkstra";

    clearRoute();
    markSelectedLocations();

    setStatus("Map reset. Select your locations.");
    $("mapStatus").textContent = "Map ready";
}

document.addEventListener("DOMContentLoaded", () => {
    populateDropdowns();
    drawRoads();
    drawLocations();
    markSelectedLocations();

    $("routeForm").addEventListener("submit", calculateRoute);

    $("swapLocations").addEventListener(
        "click",
        swapLocations
    );

    $("resetMap").addEventListener(
        "click",
        resetMap
    );

    [startSelect, destinationSelect, algorithmSelect]
        .forEach(element => {
            element.addEventListener("change", () => {
                clearRoute();
                markSelectedLocations();
            });
        });

    checkServer();

    console.log("Campus Route Finder initialized.");
});