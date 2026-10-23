from __future__ import annotations

from dataclasses import dataclass
from typing import Dict, List

try:
    import matplotlib.pyplot as plt
except ImportError:  # pragma: no cover - optional dependency
    plt = None


def clamp(value: float, low: float, high: float) -> float:
    return max(low, min(high, value))


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
        self.history: List[Dict[str, float]] = []

    def step(self, dt: float = 1.0) -> Dict[str, float]:
        """Advance the simulation by one timestep."""
        heat_stress = max(0.0, (self.env.temperature - 300.0) / 100.0)
        radiation_damage = self.env.radiation * 0.02
        desiccation = max(0.0, (1.0 - self.env.water_availability)) * 0.5

        nutrient_factor = clamp(self.env.nutrients, 0.0, 1.0)
        water_factor = clamp(self.env.water_availability, 0.0, 1.0)

        # Energy production vs depletion
        if nutrient_factor > 0.2 and water_factor > 0.2:
            net_energy_gain = (0.08 * nutrient_factor * water_factor) - (0.04 * heat_stress)
        else:
            net_energy_gain = -0.05

        self.cell.energy += net_energy_gain * dt
        self.cell.energy = clamp(self.cell.energy, 0.0, 2.0)

        # Membrane degradation
        damage_rate = heat_stress + radiation_damage + desiccation
        self.cell.damage += damage_rate * dt
        self.cell.membrane_integrity = max(0.0, 1.0 - self.cell.damage / 10.0)

        # Stress increases with harsh conditions
        self.cell.stress += (
            heat_stress
            + desiccation
            + (self.env.radiation * 0.15)
            - (self.cell.membrane_integrity * 0.15)
        ) * dt
        self.cell.stress = clamp(self.cell.stress, 0.0, 1.0)

        # Water balance
        self.cell.water_balance -= (desiccation - (0.05 * water_factor)) * dt
        self.cell.water_balance = clamp(self.cell.water_balance, 0.0, 1.2)

        # Metabolism adapts to environment and stress
        if self.cell.stress > 0.7:
            self.cell.metabolism *= 0.985
        else:
            self.cell.metabolism = clamp(
                self.cell.metabolism + 0.015 * (nutrient_factor * water_factor),
                0.0,
                1.0,
            )

        # Energy cost of metabolism
        self.cell.energy -= self.cell.metabolism * 0.02 * dt
        self.cell.energy = clamp(self.cell.energy, 0.0, 2.0)

        snapshot = {
            "time": len(self.history) * dt,
            "membrane": self.cell.membrane_integrity,
            "energy": self.cell.energy,
            "stress": self.cell.stress,
            "water": self.cell.water_balance,
            "metabolism": self.cell.metabolism,
            "damage": self.cell.damage,
        }
        self.history.append(snapshot)
        return snapshot

    def run(self, steps: int, dt: float = 1.0) -> List[Dict[str, float]]:
        for _ in range(steps):
            self.step(dt=dt)
        return self.history

    def plot_history(self, filename: str = "microbe_simulation.png", show: bool = True) -> None:
        if plt is None:
            raise ImportError("matplotlib is not installed. Install with: pip install matplotlib")

        if not self.history:
            raise ValueError("No simulation history found. Run the engine before plotting.")

        times = [entry["time"] for entry in self.history]

        fig, axes = plt.subplots(2, 3, figsize=(12, 8), constrained_layout=True)
        axes = axes.flatten()

        metrics = [
            ("membrane", "Membrane Integrity"),
            ("energy", "Cell Energy"),
            ("stress", "Cell Stress"),
            ("water", "Water Balance"),
            ("metabolism", "Metabolism"),
            ("damage", "Membrane Damage"),
        ]

        for ax, (key, label) in zip(axes, metrics):
            values = [entry[key] for entry in self.history]
            ax.plot(times, values, linewidth=2)
            ax.set_title(label)
            ax.set_xlabel("Time")
            ax.set_ylabel("Value")
            ax.grid(alpha=0.25)

        fig.suptitle("Venus/Jovian Microbe Simulation", fontsize=16)
        fig.savefig(filename, dpi=200)

        if show:
            plt.show()
        else:
            plt.close(fig)


if __name__ == "__main__":
    env = Environment(
        temperature=310.0,
        pressure=1.0,
        radiation=0.7,
        water_availability=0.6,
        nutrients=0.8,
    )
    cell = Cell(
        membrane_integrity=1.0,
        energy=1.0,
        water_balance=1.0,
        stress=0.0,
        metabolism=0.55,
        damage=0.0,
    )

    sim = SimulationEngine(env=env, cell=cell)
    history = sim.run(steps=200, dt=0.5)

    print("Final state:")
    print(history[-1])

    sim.plot_history(filename="microbe_simulation.png", show=False)
    print("Graph saved to microbe_simulation.png")
