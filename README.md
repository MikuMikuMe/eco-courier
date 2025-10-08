# eco-courier

Creating a comprehensive Python program for an eco-courier service involves numerous components such as route optimization, cost calculation, and emissions calculation. To effectively manage these tasks, we can make use of various libraries and APIs. Here's a simplified version of what such a program might look like:

```python
import requests
from ortools.constraint_solver import routing_enums_pb2
from ortools.constraint_solver import pywrapcp

# Placeholder API key for a map service like Google Maps
MAPS_API_KEY = 'YOUR_API_KEY_HERE'

def get_distance_matrix(locations):
    """Fetch the distance matrix using an external maps API."""
    # Error handling if API key is missing
    if MAPS_API_KEY == 'YOUR_API_KEY_HERE':
        raise ValueError("Missing API Key. Please set your MAPS_API_KEY.")

    matrix = []
    try:
        # Placeholder: Use a fixed distance matrix for simplicity
        # This would normally involve an API call to a service like Google Maps or Mapbox
        num_locations = len(locations)
        matrix = [[0 for _ in range(num_locations)] for _ in range(num_locations)]
        for i in range(num_locations):
            for j in range(num_locations):
                if i != j:
                    # Here we would call an API to get the distance between locations[i] and locations[j]
                    matrix[i][j] = (i + j) * 10  # Dummy distance for the example
    except Exception as e:
        print(f'Error while fetching distance matrix: {e}')
    return matrix

def optimize_route(distance_matrix):
    """Optimize delivery route using Google's OR-Tools."""
    try:
        # Create the routing index manager.
        manager = pywrapcp.RoutingIndexManager(len(distance_matrix), 1, 0)

        # Create Routing Model.
        routing = pywrapcp.RoutingModel(manager)

        def distance_callback(from_index, to_index):
            """Returns the distance between the two nodes."""
            # Convert from routing variable Index to distance matrix NodeIndex.
            from_node = manager.IndexToNode(from_index)
            to_node = manager.IndexToNode(to_index)
            return distance_matrix[from_node][to_node]

        transit_callback_index = routing.RegisterTransitCallback(distance_callback)

        # Define cost of each arc.
        routing.SetArcCostEvaluatorOfAllVehicles(transit_callback_index)

        # Set search parameters.
        search_parameters = pywrapcp.DefaultRoutingSearchParameters()
        search_parameters.first_solution_strategy = (
            routing_enums_pb2.FirstSolutionStrategy.PATH_CHEAPEST_ARC)

        # Solve the problem.
        solution = routing.SolveWithParameters(search_parameters)

    except Exception as e:
        print(f'Error during route optimization: {e}')
        return None, None

    # Extract the route
    if solution:
        route = []
        index = routing.Start(0)
        route_distance = 0
        while not routing.IsEnd(index):
            route.append(manager.IndexToNode(index))
            previous_index = index
            index = solution.Value(routing.NextVar(index))
            route_distance += routing.GetArcCostForVehicle(previous_index, index, 0)
        route.append(manager.IndexToNode(index))
        return route, route_distance
    else:
        raise ValueError("No solution found for route optimization.")

def calculate_emissions(route_distance):
    """Calculate carbon emissions based on the route distance."""
    # This is a dummy calculation. Replace with a real emissions model.
    emissions_rate_per_km = 0.2  # kg of CO2 per km
    emissions = route_distance * emissions_rate_per_km
    return emissions

def main():
    # Example set of delivery locations
    locations = ['location1', 'location2', 'location3', 'location4']

    try:
        # Get distance matrix
        distance_matrix = get_distance_matrix(locations)

        # Optimize route
        route, route_distance = optimize_route(distance_matrix)

        if route is not None:
            print('Optimized route:', route)
            print('Total distance of the route (in distance matrix units):', route_distance)

            # Calculate emissions
            emissions = calculate_emissions(route_distance)
            print(f'Total estimated emissions: {emissions:.2f} kg CO2')
        else:
            print('Route optimization failed.')
    except Exception as ex:
        print(f"An error occurred: {ex}")

if __name__ == '__main__':
    main()
```

### Key Components Explained:

1. **Distance Matrix**: Normally fetched from an API (like Google Maps), but here we use a dummy matrix as a placeholder. 

2. **Route Optimization**: Utilizes Google OR-Tools for Vehicle Routing Problem (VRP) solving which helps to optimize the delivery route based on the distance matrix.

3. **Emission Calculation**: Simple calculation based on total route distance and a predefined emissions rate per kilometer.

### Error Handling:

- Handling errors for missing API keys.
- Wrapping calls to external services or APIs inside try-except blocks to gracefully handle network errors or invalid responses.
- Providing meaningful output when a route cannot be optimized.

Before implementing in a production environment, replace placeholders with actual API integration for accurate distances, and possibly integrate with real-time traffic data for a more comprehensive solution. Also, consider refining the emissions model for more reliable carbon footprint estimation.