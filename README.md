# real-world-system
  Traffic Flow Simulation  In this example, we'll model a simple traffic flow system with a single lane road and a traffic light. We'll simulate the flow of cars through the road and observe how the traffic light affects the traffic flow.
import simpy
import random

# Define the simulation parameters
SIMULATION_TIME = 3600  # 1 hour
ROAD_CAPACITY = 100  # cars per hour
TRAFFIC_LIGHT_CYCLE = 60  # seconds

class Car:
    def __init__(self, name):
        self.name = name

class TrafficLight:
    def __init__(self, env):
        self.env = env
        self.is_green = True

    def switch(self):
        while True:
            yield self.env.timeout(TRAFFIC_LIGHT_CYCLE / 2)
            self.is_green = not self.is_green

def car_arrival(env, road, traffic_light):
    car_id = 0
    while True:
        yield env.timeout(random.expovariate(ROAD_CAPACITY / 3600))
        car = Car(f"Car {car_id}")
        car_id += 1
        print(f"{env.now:.2f}: {car.name} arrives")
        road.put(car)

def car_pass(env, road, traffic_light):
    while True:
        car = yield road.get()
        if traffic_light.is_green:
            print(f"{env.now:.2f}: {car.name} passes")
            yield env.timeout(2)  # assume 2 seconds to pass
        else:
            print(f"{env.now:.2f}: {car.name} waits")
            yield env.timeout(TRAFFIC_LIGHT_CYCLE / 2)

def run_simulation():
    env = simpy.Environment()
    road = simpy.Store(env)
    traffic_light = TrafficLight(env)

    env.process(car_arrival(env, road, traffic_light))
    env.process(car_pass(env, road, traffic_light))
    env.process(traffic_light.switch())

    env.run(until=SIMULATION_TIME)

if __name__ == "__main__":
    run_simulation()\
