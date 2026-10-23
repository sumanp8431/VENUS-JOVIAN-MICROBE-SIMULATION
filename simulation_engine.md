from dataclasses import dataclass, field

@dataclass
class Environment:
    temperature: float = 300.0
    pressure: float = 1.0
    radiation: float = 0.0
    water_availability: float = 1.0
    nutrients: float = 1.0

@dataclass
class Cell:
    membrane_integrity: float = 1.0
    energy: float = 1.0
    water_balance: float = 1.0
    stress: float = 0.0
    metabolism: float = 0.5
    damage: float = 0.0

class SimulationEngine:
    def __init__(self, env: Environment, cell: Cell):
        self.env = env
        self.cell = cell
        self.history = []

    def step(self, dt: float = 1.0):
        # environmental stress
        heat_stress = max(0.0, (self.env.temperature - 300.0) / 100.0)
        radiation_damage = self.env.radiation * 0.02
        desiccation = max(0.0, (1.0 - self.env.water_availability)) * 0.5

        # metabolism responds to environment
        if self.env.nutrients > 0.2 and self.env.water_availability > 0.2:
            self.cell.energy += (0.05 - heat_stress) * dt
        else:
            self.cell.energy -= 0.03 * dt

        # membrane degradation
        self.cell.damage += (heat_stress + radiation_damage + desiccation) * dt
        self.cell.membrane_integrity = max(0.0, 1.0 - self.cell.damage / 10.0)

        # stress increases under harsh conditions
        self.cell.stress += (heat_stress + desiccation + self.env.radiation * 0.1) * dt
        self.cell.stress = min(1.0, self.cell.stress)

        # water balance
        self.cell.water_balance -= desiccation * dt
        self.cell.water_balance = max(0.0, self.cell.water_balance)

        # if stress too high, metabolism drops
        if self.cell.stress > 0.7:
            self.cell.metabolism *= 0.98
        else:
            self.cell.metabolism = min(1.0, self.cell.metabolism + 0.01)

        # record
        self.history.append({
            "membrane": self.cell.membrane_integrity,
            "energy": self.cell.energy,
            "stress": self.cell.stress,
            "water": self.cell.water_balance,
            "metabolism": self.cell.metabolism,
        })

    def run(self, steps: int):
        for _ in range(steps):
            self.step()
        return self.history
