import numpy as np

class VenusAdaptiveSimulation:
    def __init__(self, initial_population=10000000):
        self.population = initial_population
        self.internal_temp = 37.0
        self.shield_state = "IDLE_CLEANUP"
        self.shield_density = 0.0
        self.metabolic_energy = 100.0
        self.dsup_protection_factor = 0.85
        
    def run_venus_tick(self, time_hour, current_ph, radiation, temperature, moisture, sulfur_sources):
        if self.population <= 0:
            self.population = 0
            return

        # 1. DEHYDRATION TOGGLE (Tardigrade TDP Matrix)
        # Ultra-low moisture triggers glass-like vitrification to lock cell structures
        if moisture <= 0.01:
            self.shield_state = "TUN_DORMANT"
            self.metabolic_energy = max(0.01, self.metabolic_energy - 0.002)
            vulnerability_modifier = 0.05  # Shield blocks 95% of incoming stress damage
        else:
            vulnerability_modifier = 1.0

        # 2. DUAL-TIER RADIATION RESPONSE MATRIX
        if radiation > 800 and self.shield_state != "TUN_DORMANT":
            self.shield_state = "COSMIC_SHIELD_ACTIVE"
            self.shield_density = min(1.0, radiation / 6000.0)
            unshielded_rad = radiation * (1.0 - self.shield_density)
            self.metabolic_energy += (radiation * 0.04)
        else:
            if self.shield_state != "TUN_DORMANT":
                self.shield_state = "IDLE_CLEANUP"
                self.shield_density = 0.0
            unshielded_rad = radiation * (1.0 - self.dsup_protection_factor) * vulnerability_modifier

        # 3. LITHOTROPHIC METABOLISM (The Blood Falls Feature)
        if self.shield_state != "TUN_DORMANT":
            if sulfur_sources > 0.1:
                self.metabolic_energy += (sulfur_sources * 15.0)
            else:
                self.metabolic_energy -= 8.0

        # 4. ACIDOPHILIC PROTON PUMP LOGIC (pH Maintenance)
        if current_ph < 1.0:
            # Active acid stress calculation balanced by Lactobacillus pump genes
            acid_corrosion_strain = abs(current_ph) * 0.004 * vulnerability_modifier
        else:
            acid_corrosion_strain = 0.0

        # 5. UNIFIED POPULATION VIABILITY CALCULATION
        starvation_factor = 8.0 if self.metabolic_energy <= 0 else 1.0
        
        decay_rate = (0.004 * (unshielded_rad / max(1.0, radiation)) + acid_corrosion_strain) * starvation_factor
        
        # Apply stable dual-equilibrium plateau if locked in the cloud deck
        if time_hour >= 6 and self.shield_state == "TUN_DORMANT":
            decay_rate *= 0.05  # High-efficiency survival protection factor
            
        self.population = int(self.population * (1.0 - decay_rate))

# --- VENUS ATMOSPHERIC PROFILE ARRAYS ---
hours = np.arange(25)
# Simulating a gradual descent through changing Venusian altitude zones
ph_levels = np.array([2.5 if h < 6 else -0.5 for h in hours]) # Drastic acid spike in main clouds
radiations = np.array([4000 if h < 6 else 1200 if h < 18 else 300 for h in hours]) 
temperatures = np.array([-25 if h < 6 else 65 for h in hours]) # Stratosphere chill to cloud warmth
moisture_levels = np.array([0.5 if h < 6 else 0.01 for h in hours]) # Severe desiccation entry
sulfur_availability = np.array([0.2 if h < 6 else 1.0 for h in hours]) # Thick sulfuric clouds

# Execute Venus Experiment
venus_run = VenusAdaptiveSimulation(initial_population=10000000)

print("--- EXECUTING VENUS DUAL-EQUILIBRIUM ENGINE ---")
for hour in hours:
    venus_run.run_venus_tick(hour, ph_levels[hour], radiations[hour], temperatures[hour], moisture_levels[hour], sulfur_availability[hour])
    if hour in:
        print(f"[Hour {hour:02d}] State: {venus_run.shield_state:20s} | "
              f"pH: {ph_levels[hour]:4.1f} | "
              f"Pop: {venus_run.population:10,d} cells")
