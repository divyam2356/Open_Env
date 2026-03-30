# Thermal-Ops: The Autonomous Data Center Cooling Environment

## Environment Description & Motivation
"Thermal-Ops" is an OpenEnv conformant agentic environment that simulates a task that human data center facility managers perform: managing temperature within safe thresholds (20°C - 25°C) while minimizing energy usage and electrical costs. Real-world utility (like PUE) is actively managed by cooling fans, chillers, and workload assignment.

## Observation and Action Spaces
**Observation Space**:
- `ambient_temperature`: The weather outside (simulated).
- `server_rack_temps`: A list of temperatures for different zones (T1, T2, T3).
- `current_power_load`: Heat the servers are generating (fluctuates with traffic).
- `energy_cost`: Real-time price per kWh.
- `fan_rpms`: Current cooling rates per rack.
- `chiller_setpoint`: The threshold temp for liquid cooling.
- `step_count`: The current step dimension.

**Action Space** (Agent Tools):
- `set_fan_speed(rack_id: int, rpm: int)`: High RPM cools faster but uses exponential power.
- `adjust_chiller(chiller_temp: float)`: Sets the liquid cooling setpoint.
- `migrate_workload(source_rack: int, target_rack: int)`: Moves workloads between zones.

## Task Descriptions (Difficulty Progression)
1. **Steady State (Easy)**: Maintain a constant temp while server load is static. 
2. **Heat Wave (Medium)**: Outside temperature spikes. The agent must ramp up cooling before servers throttle.
3. **Hardware Failure (Hard)**: A fan breaks down. The agent must migrate workloads to other racks to prevent melting while maintaining cost constraints.

## Requirements
- OpenEnv
- Pydantic
- OpenAI

## Setup and Usage Instructions
Install the required packages using `uv` or `pip`:
```bash
uv lock
uv pip install -e .
```
Validate the correctness:
```bash
openenv validate
```

## Baseline Scores
Running our included `baseline.py` script compares a naive "dumb baseline" (100% cooling constantly) against a "smart heuristic" strategy to emulate agent progress:

- **Steady State Task**:
  Dumb Score: `0.0` (Energy: `5000.10`) | Smart Score: `1.0` (Energy: `1305.20`)
- **Heat Wave Task**:
  Dumb Score: `0.5` (Energy: `5000.10`) | Smart Score: `1.0` (Energy: `1305.20`)
- **Hardware Failure Task**:
  Dumb Score: `0.0` (Energy: `5000.10`) | Smart Score: `1.0` (Energy: `1305.20`)
