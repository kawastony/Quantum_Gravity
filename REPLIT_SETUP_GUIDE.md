# Getting Started with Replit - Universe Simulation App

## Quick Start Guide for Building Your Universe Simulation App on Replit

This guide walks you through setting up your quantum gravity physics models on Replit and building an interactive 3D universe visualization app.

---

## Step 1: Create a Replit Account & Import Your Repo

### 1.1 Sign Up for Replit
1. Go to https://replit.com
2. Click **"Sign up"**
3. Choose **"Sign up with GitHub"** (recommended)
4. Authorize Replit to access your GitHub account

### 1.2 Import Your Quantum_Gravity Repository
1. In Replit dashboard, click **"+ Create"**
2. Click **"Import from GitHub"**
3. Paste your repo URL: `https://github.com/kawastony/Quantum_Gravity`
4. Click **"Import"**
5. Replit will clone your entire repo with all physics papers and models

---

## Step 2: Set Up the Python Environment

### 2.1 Create a Python Project Structure
In your Replit, create this directory structure:

```
quantum-gravity-app/
├── backend/
│   ├── physics_engine.py          # Your quantum gravity calculations
│   ├── nasa_integration.py        # NASA data fetching
│   └── api_server.py              # Flask/FastAPI server
├── frontend/
│   ├── index.html                 # Main web interface
│   ├── style.css                  # Styling
│   └── universe_viewer.js         # 3D visualization (Three.js)
├── requirements.txt               # Python dependencies
├── .replit                        # Replit config
└── README.md
```

### 2.2 Create `requirements.txt`
```
Flask==2.3.0
numpy==1.24.0
scipy==1.10.0
requests==2.31.0
python-dotenv==1.0.0
matplotlib==3.7.0
astropy==5.2
```

Click **"+ Add file"** in Replit and create `requirements.txt` with the above content.

---

## Step 3: Build the Physics Engine

### 3.1 Create `backend/physics_engine.py`

```python
import numpy as np
from scipy.integrate import odeint

class QuantumGravityUniverse:
    """
    Kawastony Quantum Gravity cosmological model
    Alternative to Lambda-CDM
    """
    
    def __init__(self, H0=67.2, Omega_m=0.312, Lambda_quantum=1.2e-3):
        """Initialize universe parameters"""
        self.H0 = H0  # Hubble constant (km/s/Mpc)
        self.Omega_m = Omega_m  # Matter density
        self.Lambda_quantum = Lambda_quantum  # Quantum correction (meV)
        self.Lambda_cl = 2.4e-3  # Classical cutoff
        
    def hubble_parameter(self, a):
        """
        Compute H(a) using modified Friedmann equation
        H²(a) = H₀² [Ω_m,0 a⁻³ + f_quantum(a)]
        """
        quantum_term = self.quantum_correction(a)
        H_squared = self.H0**2 * (self.Omega_m * a**(-3) + quantum_term)
        return np.sqrt(H_squared)
    
    def quantum_correction(self, a):
        """Quantum geometric correction term"""
        coefficient = self.Lambda_quantum / self.Lambda_cl
        return coefficient * (1 - np.exp(-self.Lambda_cl * a**2))
    
    def scale_factor_derivative(self, a, t):
        """da/dt = a * H(a)"""
        return a * self.hubble_parameter(a)
    
    def simulate(self, a_initial=1e-6, t_final=13.8, num_steps=1000):
        """
        Run cosmological simulation
        Returns: times, scale_factors, expansion_history
        """
        times = np.linspace(0, t_final, num_steps)
        a_values = odeint(self.scale_factor_derivative, a_initial, times)
        
        # Compute Hubble parameter at each time
        H_values = [self.hubble_parameter(a[0]) for a in a_values]
        
        # Compute redshift z = 1/a - 1
        z_values = 1.0 / a_values - 1
        
        return {
            'times': times,
            'scale_factors': a_values.flatten(),
            'hubble_parameters': H_values,
            'redshifts': z_values.flatten(),
            'universe_age': t_final
        }
    
    def get_expansion_rate(self, redshift):
        """Get Hubble parameter at given redshift"""
        a = 1.0 / (1.0 + redshift)
        return self.hubble_parameter(a)

# Example usage
if __name__ == "__main__":
    universe = QuantumGravityUniverse()
    result = universe.simulate()
    print(f"Universe age: {result['universe_age']} Gyr")
    print(f"Current Hubble: {result['hubble_parameters'][-1]:.2f} km/s/Mpc")
```

---

## Step 4: Build the API Server

### 4.1 Create `backend/api_server.py`

```python
from flask import Flask, jsonify, request
from backend.physics_engine import QuantumGravityUniverse
import json

app = Flask(__name__)

# Initialize universe simulator
universe = QuantumGravityUniverse()

@app.route('/api/models', methods=['GET'])
def get_models():
    """Get available physics models"""
    return jsonify({
        'models': [
            {
                'id': 'kawastony_lambda_cdm_alt',
                'name': 'Kawastony Alternative Lambda-CDM',
                'version': '1.0',
                'status': 'production'
            }
        ]
    })

@app.route('/api/simulate', methods=['POST'])
def simulate_universe():
    """Run universe simulation with custom parameters"""
    data = request.json
    
    # Get parameters from request (or use defaults)
    H0 = data.get('H0', 67.2)
    Omega_m = data.get('Omega_m', 0.312)
    time_steps = data.get('time_steps', 1000)
    
    # Run simulation
    sim_universe = QuantumGravityUniverse(H0=H0, Omega_m=Omega_m)
    result = sim_universe.simulate(t_final=13.8, num_steps=time_steps)
    
    # Convert numpy arrays to lists for JSON serialization
    return jsonify({
        'simulation_id': 'sim_' + str(hash(str(data)))[-8:],
        'model': 'kawastony_lambda_cdm_alt',
        'results': {
            'times': result['times'].tolist(),
            'scale_factors': result['scale_factors'].tolist(),
            'hubble_parameters': result['hubble_parameters'],
            'universe_age_gyr': result['universe_age'],
            'final_hubble': float(result['hubble_parameters'][-1])
        },
        'status': 'completed'
    })

@app.route('/api/hubble/<float:redshift>', methods=['GET'])
def get_hubble_at_redshift(redshift):
    """Get Hubble parameter at specific redshift"""
    H = universe.get_expansion_rate(redshift)
    return jsonify({
        'redshift': redshift,
        'hubble_parameter': float(H),
        'unit': 'km/s/Mpc'
    })

@app.route('/api/visualization/<simulation_id>', methods=['GET'])
def get_visualization_data(simulation_id):
    """Get data for 3D visualization"""
    # Run default simulation
    result = universe.simulate()
    
    return jsonify({
        'simulation_id': simulation_id,
        'visualization_data': {
            'times': result['times'].tolist()[:100],  # Downsample for web
            'scale_factors': result['scale_factors'].tolist()[:100],
            'hubble_parameters': result['hubble_parameters'][:100]
        }
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3000, debug=True)
```

---

## Step 5: Build the Frontend with 3D Visualization

### 5.1 Create `frontend/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quantum Gravity Universe Simulator</title>
    <link rel="stylesheet" href="style.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <div class="container">
        <header>
            <h1>🌌 Quantum Gravity Universe Simulator</h1>
            <p>Interactive visualization of the Kawastony Alternative to Lambda-CDM</p>
        </header>
        
        <div class="controls">
            <div class="control-group">
                <label for="h0-slider">Hubble Constant (H₀): <span id="h0-value">67.2</span> km/s/Mpc</label>
                <input type="range" id="h0-slider" min="65" max="70" step="0.1" value="67.2">
            </div>
            
            <div class="control-group">
                <label for="omega-slider">Matter Density (Ω_m): <span id="omega-value">0.312</span></label>
                <input type="range" id="omega-slider" min="0.28" max="0.35" step="0.001" value="0.312">
            </div>
            
            <button id="simulate-btn" class="btn btn-primary">Run Simulation</button>
            <button id="compare-btn" class="btn btn-secondary">Compare to Lambda-CDM</button>
        </div>
        
        <div class="content">
            <div class="visualization">
                <h2>Universe Expansion History</h2>
                <canvas id="hubble-chart"></canvas>
            </div>
            
            <div class="info-panel">
                <h2>Model Parameters</h2>
                <div id="model-info"></div>
                
                <h2>Simulation Results</h2>
                <div id="simulation-results"></div>
            </div>
        </div>
        
        <div class="nasa-integration">
            <h2>NASA Data Integration</h2>
            <button id="load-hubble" class="btn">Load Hubble Data</button>
            <button id="load-gaia" class="btn">Load Gaia Stars</button>
            <button id="load-sdss" class="btn">Load SDSS Galaxies</button>
            <div id="nasa-data"></div>
        </div>
    </div>
    
    <script src="universe_viewer.js"></script>
</body>
</html>
```

### 5.2 Create `frontend/style.css`

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #0a0e27 0%, #1a1a3e 100%);
    color: #e0e0e0;
    min-height: 100vh;
    padding: 20px;
}

.container {
    max-width: 1400px;
    margin: 0 auto;
}

header {
    text-align: center;
    margin-bottom: 40px;
}

header h1 {
    font-size: 2.5em;
    color: #00d4ff;
    margin-bottom: 10px;
    text-shadow: 0 0 20px rgba(0, 212, 255, 0.5);
}

header p {
    font-size: 1.1em;
    color: #b0b0b0;
}

.controls {
    background: rgba(255, 255, 255, 0.05);
    border: 2px solid rgba(0, 212, 255, 0.3);
    border-radius: 10px;
    padding: 20px;
    margin-bottom: 30px;
    backdrop-filter: blur(10px);
}

.control-group {
    margin-bottom: 15px;
}

.control-group label {
    display: block;
    margin-bottom: 8px;
    font-weight: 500;
    color: #00d4ff;
}

.control-group input[type="range"] {
    width: 100%;
    height: 6px;
    cursor: pointer;
}

.btn {
    padding: 12px 24px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1em;
    font-weight: 600;
    transition: all 0.3s ease;
    margin-right: 10px;
    margin-top: 10px;
}

.btn-primary {
    background: linear-gradient(135deg, #00d4ff, #0099cc);
    color: white;
}

.btn-primary:hover {
    transform: translateY(-2px);
    box-shadow: 0 10px 20px rgba(0, 212, 255, 0.3);
}

.btn-secondary {
    background: rgba(0, 212, 255, 0.2);
    color: #00d4ff;
    border: 2px solid #00d4ff;
}

.btn-secondary:hover {
    background: rgba(0, 212, 255, 0.3);
}

.content {
    display: grid;
    grid-template-columns: 2fr 1fr;
    gap: 20px;
    margin-bottom: 30px;
}

.visualization, .info-panel {
    background: rgba(255, 255, 255, 0.05);
    border: 2px solid rgba(0, 212, 255, 0.3);
    border-radius: 10px;
    padding: 20px;
    backdrop-filter: blur(10px);
}

.visualization h2, .info-panel h2 {
    color: #00d4ff;
    margin-bottom: 15px;
    font-size: 1.3em;
}

canvas {
    width: 100%;
    height: auto;
}

#model-info, #simulation-results {
    font-size: 0.95em;
    line-height: 1.8;
}

#model-info p, #simulation-results p {
    margin-bottom: 10px;
}

.nasa-integration {
    background: rgba(255, 255, 255, 0.05);
    border: 2px solid rgba(0, 212, 255, 0.3);
    border-radius: 10px;
    padding: 20px;
    backdrop-filter: blur(10px);
}

.nasa-integration h2 {
    color: #00d4ff;
    margin-bottom: 15px;
}

#nasa-data {
    margin-top: 15px;
    max-height: 200px;
    overflow-y: auto;
}

@media (max-width: 768px) {
    .content {
        grid-template-columns: 1fr;
    }
    
    header h1 {
        font-size: 1.8em;
    }
}
```

### 5.3 Create `frontend/universe_viewer.js`

```javascript
// Initialize Chart.js for expansion history
let hubbleChart;

document.addEventListener('DOMContentLoaded', () => {
    initializeChart();
    setupEventListeners();
});

function initializeChart() {
    const ctx = document.getElementById('hubble-chart').getContext('2d');
    hubbleChart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: [],
            datasets: [
                {
                    label: 'Kawastony Model',
                    data: [],
                    borderColor: '#00d4ff',
                    backgroundColor: 'rgba(0, 212, 255, 0.1)',
                    tension: 0.3,
                    fill: true
                },
                {
                    label: 'Lambda-CDM',
                    data: [],
                    borderColor: '#ff9500',
                    backgroundColor: 'rgba(255, 149, 0, 0.1)',
                    tension: 0.3,
                    fill: true
                }
            ]
        },
        options: {
            responsive: true,
            plugins: {
                legend: {
                    labels: { color: '#e0e0e0' }
                }
            },
            scales: {
                x: {
                    title: { display: true, text: 'Time (Gyr)', color: '#e0e0e0' },
                    ticks: { color: '#b0b0b0' },
                    grid: { color: 'rgba(255, 255, 255, 0.1)' }
                },
                y: {
                    title: { display: true, text: 'Hubble Parameter (km/s/Mpc)', color: '#e0e0e0' },
                    ticks: { color: '#b0b0b0' },
                    grid: { color: 'rgba(255, 255, 255, 0.1)' }
                }
            }
        }
    });
}

function setupEventListeners() {
    document.getElementById('h0-slider').addEventListener('input', (e) => {
        document.getElementById('h0-value').textContent = e.target.value;
    });
    
    document.getElementById('omega-slider').addEventListener('input', (e) => {
        document.getElementById('omega-value').textContent = e.target.value;
    });
    
    document.getElementById('simulate-btn').addEventListener('click', runSimulation);
    document.getElementById('compare-btn').addEventListener('click', compareModels);
}

async function runSimulation() {
    const H0 = parseFloat(document.getElementById('h0-slider').value);
    const Omega_m = parseFloat(document.getElementById('omega-slider').value);
    
    try {
        const response = await fetch('/api/simulate', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                H0: H0,
                Omega_m: Omega_m,
                time_steps: 100
            })
        });
        
        const data = await response.json();
        updateVisualization(data.results);
        updateModelInfo(H0, Omega_m);
        updateSimulationResults(data.results);
        
    } catch (error) {
        console.error('Simulation error:', error);
        alert('Error running simulation. Check console.');
    }
}

function updateVisualization(results) {
    hubbleChart.data.labels = results.times.map(t => t.toFixed(1));
    hubbleChart.data.datasets[0].data = results.hubble_parameters;
    hubbleChart.update();
}

function updateModelInfo(H0, Omega_m) {
    const info = document.getElementById('model-info');
    info.innerHTML = `
        <p><strong>Model:</strong> Kawastony Alternative Lambda-CDM</p>
        <p><strong>H₀:</strong> ${H0} km/s/Mpc</p>
        <p><strong>Ω_m:</strong> ${Omega_m}</p>
        <p><strong>Status:</strong> Production ✓</p>
    `;
}

function updateSimulationResults(results) {
    const results_div = document.getElementById('simulation-results');
    results_div.innerHTML = `
        <p><strong>Universe Age:</strong> ${results.universe_age_gyr.toFixed(2)} Gyr</p>
        <p><strong>Current Hubble:</strong> ${results.final_hubble.toFixed(2)} km/s/Mpc</p>
        <p><strong>Time Steps:</strong> ${results.times.length}</p>
    `;
}

async function compareModels() {
    alert('Model comparison feature coming soon!');
}
```

---

## Step 6: Configure Replit

### 6.1 Create `.replit` Configuration File

```
language = "python3"
run = "python backend/api_server.py"

[nix]
channel = "stable-22.05"

[[ports]]
localPort = 3000
externalPort = 80
```

### 6.2 Create `.env` File (for NASA API keys)

```
NASA_API_KEY=your_nasa_api_key_here
GITHUB_REPO=https://github.com/kawastony/Quantum_Gravity
```

---

## Step 7: Deploy & Run on Replit

### 7.1 Push Your Code to Replit

1. In Replit, click **"Git"** → **"GitHub"**
2. Select **"Push"** to sync your local changes
3. Or manually create files in Replit interface

### 7.2 Run Your App

1. Click **"Run"** button (top center)
2. Replit will install dependencies and start the Flask server
3. Your app will be live at: `https://your-username.replit.dev`

### 7.3 Test the API

Visit in your browser:
- `https://your-username.replit.dev/` → Frontend
- `https://your-username.replit.dev/api/models` → Available models
- `https://your-username.replit.dev/api/hubble/0` → Hubble at redshift 0

---

## Step 8: Add NASA Data Integration

### Create `backend/nasa_integration.py`

```python
import requests
import os

class NASADataIntegration:
    def __init__(self):
        self.api_key = os.getenv('NASA_API_KEY', 'DEMO_KEY')
        self.hubble_url = "https://archive.stsci.edu/hst/search.php"
        self.gaia_url = "https://gea.esac.esa.int/tap-server/tap/sync"
        
    def get_hubble_images(self, ra, dec, radius=0.1):
        """Fetch Hubble images near coordinates"""
        params = {
            'action': 'Search',
            'RA': ra,
            'DEC': dec,
            'radius': radius,
            'max_records': 10
        }
        try:
            response = requests.get(self.hubble_url, params=params)
            return response.json()
        except Exception as e:
            return {'error': str(e)}
    
    def get_gaia_stars(self, ra, dec, radius=1.0):
        """Fetch Gaia star data"""
        query = f"""
        SELECT TOP 100 source_id, ra, dec, parallax, phot_g_mean_mag
        FROM gaiaedr3.gaia_source
        WHERE CONTAINS(POINT(ra, dec), CIRCLE({ra}, {dec}, {radius})) = 1
        """
        return {'query': query, 'status': 'ready'}
    
    def get_sdss_galaxies(self, ra, dec, radius=0.5):
        """Fetch SDSS galaxy data"""
        return {
            'source': 'SDSS',
            'coordinates': {'ra': ra, 'dec': dec, 'radius': radius},
            'status': 'ready'
        }
```

---

## Step 9: Next Steps & Enhancements

### 🔄 Add to Your GitHub

To sync back to GitHub:
```bash
git add .
git commit -m "Add Replit universe simulator with physics engine"
git push origin main
```

### 🌟 Planned Features

- [ ] 3D universe visualization with Three.js
- [ ] Real-time NASA data overlay
- [ ] Model comparison tools
- [ ] Export simulation results
- [ ] Share simulations via URL
- [ ] Mobile app version

### 🚀 Deployment

When ready to go live:
1. Upgrade Replit to **Replit Pro** ($7/month)
2. Increase resource limits
3. Add custom domain
4. Set up CI/CD with GitHub

---

## Support & Resources

- **Replit Docs:** https://docs.replit.com
- **Three.js Docs:** https://threejs.org/docs
- **Flask Docs:** https://flask.palletsprojects.com
- **Your Physics Models:** `/papers` directory in your repo

---

**You're all set!** 🚀 Start building on Replit now!
