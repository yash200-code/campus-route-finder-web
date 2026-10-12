#define WIN32_LEAN_AND_MEAN
#include <winsock2.h>
#include <iostream>
#include <string>
#include <vector>
#include <queue>
#include <algorithm>
#include <limits>
#include <functional>
#include <sstream>

struct Edge {
    int to, weight;
};

struct Location {
    int id;
    std::string name;
    int x, y;
};

std::vector<Location> locations = {
    {1, "Security Office", 390, 510},
    {2, "Parking", 390, 445},
    {3, "Administrative Block", 465, 410},
    {4, "Library", 330, 405},
    {5, "Learning Centre 2", 245, 355},
    {6, "Halls of Residence", 620, 105},
    {7, "Sports Grounds", 390, 270},
    {8, "Learning Centre 1", 490, 230},
    {9, "Temple", 390, 110},
    {10, "Play Courts", 670, 330},
    {11, "Food Court", 555, 290},
    {12, "University Parking", 760, 130},
    {13, "Student Activity Centre", 525, 330}
};

std::vector<std::vector<Edge>> graph(14);

void addRoad(int a, int b, int distance) {
    graph[a].push_back({b, distance});
    graph[b].push_back({a, distance});
}

void buildGraph() {
    addRoad(1, 2, 65);
    addRoad(2, 3, 95);
    addRoad(2, 4, 75);
    addRoad(4, 5, 110);
    addRoad(3, 7, 145);
    addRoad(2, 7, 175);
    addRoad(7, 8, 120);
    addRoad(8, 9, 155);
    addRoad(8, 6, 170);
    addRoad(8, 11, 95);
    addRoad(11, 10, 115);
    addRoad(10, 13, 55);
    addRoad(13, 3, 105);
    addRoad(10, 6, 120);
    addRoad(6, 12, 135);
    addRoad(11, 12, 210);
    addRoad(7, 9, 190);
    addRoad(7, 13, 150);
}

struct Result {
    std::vector<int> path;
    int distance = -1;
};

// BFS: finds a route with the fewest road segments.
Result BFS(int start, int destination) {
    std::vector<int> parent(14, -1);
    std::vector<int> visited(14, 0);
    std::queue<int> q;

    q.push(start);
    visited[start] = 1;

    while (!q.empty()) {
        int u = q.front();
        q.pop();

        if (u == destination)
            break;

        for (auto edge : graph[u]) {
            if (!visited[edge.to]) {
                visited[edge.to] = 1;
                parent[edge.to] = u;
                q.push(edge.to);
            }
        }
    }

    Result result;

    if (!visited[destination])
        return result;

    for (int v = destination; v != -1; v = parent[v])
        result.path.push_back(v);

    std::reverse(result.path.begin(), result.path.end());
    result.distance = static_cast<int>(result.path.size()) - 1;

    return result;
}

// DFS: finds a path using depth-first traversal.
Result DFS(int start, int destination) {
    std::vector<int> parent(14, -1);
    std::vector<int> visited(14, 0);
    std::vector<int> stack;

    stack.push_back(start);
    visited[start] = 1;

    while (!stack.empty()) {
        int u = stack.back();
        stack.pop_back();

        if (u == destination)
            break;

        for (auto it = graph[u].rbegin();
             it != graph[u].rend(); ++it) {

            int v = it->to;

            if (!visited[v]) {
                visited[v] = 1;
                parent[v] = u;
                stack.push_back(v);
            }
        }
    }

    Result result;

    if (!visited[destination])
        return result;

    for (int v = destination; v != -1; v = parent[v])
        result.path.push_back(v);

    std::reverse(result.path.begin(), result.path.end());

    result.distance = 0;

    for (size_t i = 1; i < result.path.size(); i++) {
        int a = result.path[i - 1];
        int b = result.path[i];

        for (auto edge : graph[a]) {
            if (edge.to == b) {
                result.distance += edge.weight;
                break;
            }
        }
    }

    return result;
}

// Dijkstra: finds the minimum-distance route.
Result Dijkstra(int start, int destination) {
    const int INF = std::numeric_limits<int>::max() / 4;

    std::vector<int> distance(14, INF);
    std::vector<int> parent(14, -1);

    using Pair = std::pair<int, int>;

    std::priority_queue<
        Pair,
        std::vector<Pair>,
        std::greater<Pair>
    > pq;

    distance[start] = 0;
    pq.push({0, start});

    while (!pq.empty()) {
        auto current = pq.top();
        pq.pop();

        int d = current.first;
        int u = current.second;

        if (d != distance[u])
            continue;

        if (u == destination)
            break;

        for (auto edge : graph[u]) {
            int v = edge.to;
            int newDistance = distance[u] + edge.weight;

            if (newDistance < distance[v]) {
                distance[v] = newDistance;
                parent[v] = u;
                pq.push({newDistance, v});
            }
        }
    }

    Result result;

    if (distance[destination] == INF)
        return result;

    for (int v = destination; v != -1; v = parent[v])
        result.path.push_back(v);

    std::reverse(result.path.begin(), result.path.end());

    result.distance = distance[destination];

    return result;
}

std::string jsonEscape(const std::string& text) {
    std::string result;

    for (char c : text) {
        if (c == '"' || c == '\\')
            result += '\\';

        if (c != '\r' && c != '\n')
            result += c;
    }

    return result;
}

std::string makeResponse(
    int start,
    int destination,
    const std::string& algorithm
) {
    if (start < 1 || start > 13 ||
        destination < 1 || destination > 13) {
        return R"({"error":"Invalid location selected."})";
    }

    if (start == destination) {
        return R"({"error":"Choose different starting and destination locations."})";
    }

    Result result;
    std::string selectedAlgorithm = algorithm;

    if (algorithm == "bfs") {
        result = BFS(start, destination);
    } else if (algorithm == "dfs") {
        result = DFS(start, destination);
    } else {
        selectedAlgorithm = "dijkstra";
        result = Dijkstra(start, destination);
    }

    if (result.path.empty())
        return R"({"error":"No route found."})";

    std::ostringstream out;

    out << "{\"start\":" << start
        << ",\"destination\":" << destination
        << ",\"algorithm\":\"" << selectedAlgorithm << "\""
        << ",\"distance\":" << result.distance
        << ",\"distanceUnit\":\""
        << (selectedAlgorithm == "bfs" ? "edges" : "m")
        << "\",\"path\":[";

    for (size_t i = 0; i < result.path.size(); i++) {
        if (i > 0)
            out << ",";

        int id = result.path[i];
        const auto& location = locations[id - 1];

        out << "{\"id\":" << id
            << ",\"name\":\"" << jsonEscape(location.name) << "\""
            << ",\"x\":" << location.x
            << ",\"y\":" << location.y << "}";
    }

    out << "]}";

    return out.str();
}

void sendResponse(SOCKET client, int status, const std::string& body) {
    std::string statusText = status == 200 ? "OK" : "Bad Request";

    std::ostringstream response;

    response << "HTTP/1.1 " << status << " " << statusText << "\r\n"
             << "Content-Type: application/json; charset=utf-8\r\n"
             << "Access-Control-Allow-Origin: *\r\n"
             << "Access-Control-Allow-Methods: GET, OPTIONS\r\n"
             << "Access-Control-Allow-Headers: Content-Type\r\n"
             << "Connection: close\r\n"
             << "Content-Length: " << body.size() << "\r\n\r\n"
             << body;

    std::string message = response.str();

    send(client, message.c_str(),
         static_cast<int>(message.size()), 0);
}

int getParameter(
    const std::string& request,
    const std::string& key,
    int fallback
) {
    std::string token = key + "=";
    size_t pos = request.find(token);

    if (pos == std::string::npos)
        return fallback;

    pos += token.size();

    size_t end = request.find_first_of("& \r\n", pos);

    try {
        return std::stoi(
            request.substr(
                pos,
                end == std::string::npos
                    ? end
                    : end - pos
            )
        );
    } catch (...) {
        return fallback;
    }
}

std::string getStringParameter(
    const std::string& request,
    const std::string& key,
    const std::string& fallback
) {
    std::string token = key + "=";
    size_t pos = request.find(token);

    if (pos == std::string::npos)
        return fallback;

    pos += token.size();

    size_t end = request.find_first_of("& \r\n", pos);

    return request.substr(
        pos,
        end == std::string::npos
            ? end
            : end - pos
    );
}

int main() {
    buildGraph();

    WSADATA wsa;

    if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0) {
        std::cerr << "Winsock initialization failed.\n";
        return 1;
    }

    SOCKET server = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);

    if (server == INVALID_SOCKET) {
        std::cerr << "Could not create server socket.\n";
        WSACleanup();
        return 1;
    }

    sockaddr_in address{};
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = htonl(INADDR_LOOPBACK);
    address.sin_port = htons(8080);

    if (bind(server, reinterpret_cast<sockaddr*>(&address),
             sizeof(address)) == SOCKET_ERROR) {
        std::cerr << "Port 8080 is already in use.\n";
        closesocket(server);
        WSACleanup();
        return 1;
    }

    if (listen(server, SOMAXCONN) == SOCKET_ERROR) {
        std::cerr << "Server could not start listening.\n";
        closesocket(server);
        WSACleanup();
        return 1;
    }

    std::cout << "Campus Route Finder running at "
              << "http://localhost:8080\n";
    std::cout << "Keep this terminal open.\n";

    while (true) {
        SOCKET client = accept(server, nullptr, nullptr);

        if (client == INVALID_SOCKET)
            continue;

        char buffer[8192]{};

        int bytes = recv(client, buffer, sizeof(buffer) - 1, 0);

        if (bytes > 0) {
            std::string request(buffer, bytes);

            if (request.rfind("OPTIONS ", 0) == 0) {
                sendResponse(client, 200, "{}");
            } else if (request.find("GET /health") == 0) {
                sendResponse(
                    client, 200,
                    R"({"ok":true,"service":"Campus Route Finder"})"
                );
            } else if (request.find("GET /route") == 0) {
                int start = getParameter(request, "start", -1);
                int destination = getParameter(request, "destination", -1);

                std::string algorithm =
                    getStringParameter(request, "algorithm", "dijkstra");

                std::string body =
                    makeResponse(start, destination, algorithm);

                int status =
                    body.find("\"error\"") == std::string::npos
                    ? 200 : 400;

                sendResponse(client, status, body);
            } else {
                sendResponse(
                    client, 404,
                    R"({"error":"Unknown endpoint."})"
                );
            }
        }

        shutdown(client, SD_SEND);
        closesocket(client);
    }

    closesocket(server);
    WSACleanup();

    return 0;
}