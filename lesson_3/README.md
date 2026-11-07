# Lesson 3: Cargo delivery in Python

This exercise models cargo delivery between warehouses. A delivery can go
straight to its destination or stop at intermediate warehouses. Each vehicle
type defines which cargo it can carry and how the cargo affects its rate.

## Run the example

From the repository root, run the script with Python 3. It uses the standard
library.

```bash
python3 lesson_3/dz.py
```

The example sends electronics from Gomel to Minsk and frozen goods along a
route that stops in Brest. It prints the distance, cost, and estimated travel
time for each delivery, then starts the first one.

## Class diagram

```mermaid
classDiagram
    direction LR

    class TransportationSystem {
        +create_delivery()
        +find_suitable_transports()
        +get_delivery_status()
        +start_delivery() bool
        +complete_delivery() bool
    }
    class Delivery {
        +distance
        +cost
        +estimated_time
        +status
        +calculate_delivery_details()
        +get_route_info()
    }
    class Warehouse {
        +id
        +name
        +address
    }
    class Cargo {
        +weight
        +volume
        +requires_refrigeration
        +fragile
        +dangerous
    }
    class CargoType {
        <<enumeration>>
    }
    class Transport {
        <<abstract>>
        +max_weight
        +max_volume
        +cost_per_km
        +speed
        +can_transport() bool
        +calculate_cost_factor() float
    }
    class TransportType {
        <<enumeration>>
    }
    class Truck
    class RefrigeratorTruck {
        +min_temp
    }
    class AirTransport
    class DistanceCalculator {
        +calculate_distance() float
    }
    class CostCalculator {
        +calculate_cost() float
        +calculate_time() timedelta
    }

    Transport <|-- Truck
    Transport <|-- RefrigeratorTruck
    Transport <|-- AirTransport
    Transport --> TransportType
    Cargo --> CargoType
    TransportationSystem "1" --> "0..*" Warehouse : registers
    TransportationSystem "1" --> "0..*" Transport : registers
    TransportationSystem "1" --> "0..*" Cargo : registers
    TransportationSystem "1" --> "0..*" Delivery : tracks
    Delivery --> "1" Cargo : carries
    Delivery --> "1" Transport : uses
    Delivery --> "1" Warehouse : origin
    Delivery --> "1" Warehouse : destination
    Delivery --> "0..*" Warehouse : stops
    Delivery ..> DistanceCalculator
    Delivery ..> CostCalculator
```

## How a delivery works

1. The system looks up the cargo, vehicle, and warehouses by ID. An unknown
   warehouse ID or an incompatible vehicle prevents the delivery from being
   registered.
2. The delivery keeps one simulated distance for its route. The cost, travel
   time, and printed distance all use that value. The time estimate includes
   one hour for handling.
3. A delivery starts as `pending`, moves to `in_transit` when it departs, and
   becomes `delivered` when it arrives. An invalid or repeated transition
   returns `False` and preserves the recorded timestamps.
