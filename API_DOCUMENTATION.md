# API Documentation - Quantum Gravity Physics Models

## Overview

This API provides programmatic access to the Kawastony Quantum Gravity physics models and computational simulations. Integrate advanced cosmological simulations into your applications with a simple REST interface or Python SDK.

**Base URL:** `https://api.quantum-gravity.io/v1`

---

## Quick Start

### REST API Example

```bash
# Get available physics models
curl -X GET "https://api.quantum-gravity.io/v1/models" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Run a universe simulation
curl -X POST "https://api.quantum-gravity.io/v1/simulate/universe" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "kawastony_lambda_cdm_alt",
    "time_steps": 1000,
    "scale": "mega_parsec"
  }'
```

### Python SDK Example

```python
from quantum_gravity import UniverseSimulator, NASADataIntegration

# Initialize simulator with your physics model
simulator = UniverseSimulator(
    model="kawastony_lambda_cdm_alt",
    api_key="YOUR_API_KEY"
)

# Integrate NASA observational data
nasa_data = NASADataIntegration(
    datasets=["hubble", "gaia", "sdss"],
    api_key="YOUR_NASA_API_KEY"
)

# Run simulation
result = simulator.simulate(
    time_steps=1000,
    include_observations=nasa_data,
    output_format="3d_visualization"
)

# Render visualization
result.visualize()
```

---

## Authentication

All API requests require authentication via API key.

### Get an API Key

1. Register at: `https://quantum-gravity.io/auth/register`
2. Create API key in dashboard: `https://quantum-gravity.io/dashboard/keys`
3. Include key in request headers:

```
Authorization: Bearer sk_live_xxxxxxxxxxxxx
```

### Rate Limits

- **Free Tier:** 100 requests/hour
- **Professional:** 10,000 requests/hour
- **Enterprise:** Unlimited (custom agreement)

---

## Endpoints

### 1. Models

#### GET /models
Retrieve available physics models.

**Request:**
```bash
GET /models
```

**Response:**
```json
{
  "models": [
    {
      "id": "kawastony_lambda_cdm_alt",
      "name": "Kawastony Alternative Lambda-CDM",
      "description": "Novel quantum gravity approach to cosmology",
      "version": "1.0",
      "status": "production",
      "equations": [
        "ds² = -c²dt² + a(t)² dΩ²",
        "H(a) = √(ρ_total/3M_pl²) - quantum_corrections"
      ],
      "parameters": {
        "scale_factor": "double",
        "hubble_parameter": "double",
        "quantum_corrections": "boolean"
      }
    }
  ],
  "total": 1
}
```

#### GET /models/{model_id}
Get detailed specifications for a specific model.

**Request:**
```bash
GET /models/kawastony_lambda_cdm_alt
```

**Response:**
```json
{
  "id": "kawastony_lambda_cdm_alt",
  "name": "Kawastony Alternative Lambda-CDM",
  "description": "...",
  "documentation_url": "https://github.com/kawastony/Quantum_Gravity/blob/main/papers/README.md",
  "citation": {
    "authors": ["Tony Kawas"],
    "year": 2026,
    "repository": "https://github.com/kawastony/Quantum_Gravity"
  },
  "parameters": { ... },
  "validation": {
    "hubble_constant_prediction": "67.4 ± 0.5 km/s/Mpc",
    "cmb_predictions": "validated",
    "matter_power_spectrum": "validated"
  }
}
```

---

### 2. Simulations

#### POST /simulate/universe
Run a full universe simulation with your specified parameters.

**Request:**
```bash
POST /simulate/universe
Content-Type: application/json

{
  "model": "kawastony_lambda_cdm_alt",
  "time_steps": 1000,
  "scale": "mega_parsec",
  "initial_conditions": {
    "redshift": 1100,
    "temperature": 2973
  },
  "quantum_corrections": true,
  "include_dark_matter": true,
  "include_dark_energy": false,
  "output_format": "json"
}
```

**Response:**
```json
{
  "simulation_id": "sim_abc123xyz",
  "model": "kawastony_lambda_cdm_alt",
  "status": "completed",
  "execution_time_ms": 5234,
  "results": {
    "final_scale_factor": 1.0,
    "final_hubble_constant": 67.4,
    "universe_age_gyr": 13.8,
    "structure_formation": {
      "galaxies_formed": 2000000000000,
      "dark_matter_clusters": 89234
    },
    "timeline": [
      {
        "time_step": 0,
        "age_gyr": 0.0,
        "scale_factor": 0.001,
        "matter_density": 1e27
      },
      { ... more timesteps ... }
    ]
  },
  "visualization_url": "https://api.quantum-gravity.io/visualizations/sim_abc123xyz"
}
```

#### GET /simulate/{simulation_id}
Retrieve results from a previous simulation.

**Request:**
```bash
GET /simulate/sim_abc123xyz
```

**Response:**
```json
{
  "simulation_id": "sim_abc123xyz",
  "status": "completed",
  "results": { ... }
}
```

#### POST /simulate/compare
Compare predictions from multiple models.

**Request:**
```bash
POST /simulate/compare
Content-Type: application/json

{
  "models": [
    "kawastony_lambda_cdm_alt",
    "standard_lambda_cdm"
  ],
  "parameters": {
    "time_steps": 500,
    "scale": "mega_parsec"
  }
}
```

**Response:**
```json
{
  "comparison_id": "cmp_def456",
  "models_compared": 2,
  "differences": {
    "structure_formation_rate": "+12.3%",
    "dark_matter_abundance": "-5.1%",
    "expansion_history": "0.1% variance"
  }
}
```

---

### 3. NASA Data Integration

#### GET /data/nasa/hubble
Fetch Hubble Space Telescope observational data.

**Request:**
```bash
GET /data/nasa/hubble?region=deep_field&resolution=high
```

**Response:**
```json
{
  "dataset": "hubble_deep_field",
  "images": [
    {
      "id": "hudf_001",
      "wavelength_nm": 600,
      "resolution": "1600x1600",
      "magnitude_limit": 31.5,
      "data_url": "https://api.quantum-gravity.io/data/hudf_001.fits"
    }
  ]
}
```

#### GET /data/nasa/gaia
Fetch Gaia star catalog data.

**Request:**
```bash
GET /data/nasa/gaia?distance_pc=1000&magnitude_limit=15
```

**Response:**
```json
{
  "dataset": "gaia_dr3",
  "stars": [
    {
      "gaia_id": "Gaia DR3 123456789",
      "ra": 123.456,
      "dec": -45.678,
      "parallax_mas": 1.234,
      "distance_pc": 812.0,
      "magnitude_v": 12.5
    }
  ],
  "total_records": 1847293
}
```

#### GET /data/nasa/sdss
Fetch SDSS galaxy survey data.

**Request:**
```bash
GET /data/nasa/sdss?redshift_min=0&redshift_max=0.1
```

**Response:**
```json
{
  "dataset": "sdss_galaxies",
  "galaxies": [
    {
      "sdss_id": "1237645943812768256",
      "ra": 135.0,
      "dec": 12.5,
      "redshift": 0.05,
      "magnitude_r": 15.2,
      "galaxy_type": "spiral"
    }
  ],
  "total_records": 934000
}
```

---

### 4. Visualization

#### GET /visualize/{simulation_id}
Generate interactive 3D visualization of simulation results.

**Request:**
```bash
GET /visualize/sim_abc123xyz?format=webgl&camera_position=default
```

**Response:**
```json
{
  "visualization_url": "https://visualize.quantum-gravity.io/sim_abc123xyz",
  "format": "webgl",
  "interactive_features": [
    "pan",
    "zoom",
    "time_scrubber",
    "model_overlay",
    "data_filter"
  ],
  "embed_code": "<iframe src='https://visualize.quantum-gravity.io/...'></iframe>"
}
```

---

## Error Handling

### Error Response Format

```json
{
  "error": {
    "code": "INVALID_MODEL",
    "message": "Model 'unknown_model' not found",
    "details": "Available models: kawastony_lambda_cdm_alt",
    "timestamp": "2026-06-23T01:30:00Z"
  }
}
```

### Common Error Codes

| Code | Status | Meaning |
|------|--------|---------|
| INVALID_MODEL | 400 | Requested model doesn't exist |
| INVALID_PARAMETERS | 400 | Simulation parameters invalid |
| AUTH_FAILED | 401 | API key invalid or expired |
| RATE_LIMIT_EXCEEDED | 429 | Too many requests |
| SERVER_ERROR | 500 | Internal server error |

---

## Examples

### Example 1: Compare Universe Evolution

```python
from quantum_gravity import UniverseSimulator

simulator = UniverseSimulator(api_key="sk_live_xxx")

# Kawastony model
result1 = simulator.simulate(
    model="kawastony_lambda_cdm_alt",
    time_steps=1000
)

# Standard Lambda-CDM
result2 = simulator.simulate(
    model="standard_lambda_cdm",
    time_steps=1000
)

# Compare
differences = result1.compare_with(result2)
print(f"Hubble constant difference: {differences['hubble_constant_percent']}%")
```

### Example 2: Overlay with NASA Data

```python
from quantum_gravity import UniverseSimulator, NASADataIntegration

simulator = UniverseSimulator(api_key="sk_live_xxx")
nasa = NASADataIntegration(
    datasets=["gaia", "sdss"],
    api_key="YOUR_NASA_KEY"
)

result = simulator.simulate(
    model="kawastony_lambda_cdm_alt",
    time_steps=500,
    overlay_data=nasa.get_large_scale_structure()
)

result.visualize(show_observations=True)
```

### Example 3: Build Custom UI

```html
<!DOCTYPE html>
<html>
<head>
    <script src="https://api.quantum-gravity.io/js/visualizer.js"></script>
</head>
<body>
    <div id="universe-viewer"></div>
    
    <script>
        const viewer = new UniverseVisualizer({
            container: '#universe-viewer',
            simulationId: 'sim_abc123xyz',
            apiKey: 'sk_live_xxx'
        });
        
        viewer.on('timeChanged', (time) => {
            console.log(`Universe age: ${time} Gyr`);
        });
    </script>
</body>
</html>
```

---

## Licensing & Commercial Use

### Academic License
- Free for research and education
- Cite the original papers
- No commercial use

### Commercial License
- Contact: kawastony2@gmail.com or tkawas@scpng.gov.pg
- Revenue sharing options available
- API usage unlimited
- Priority support

---

## Support & Contact

**Documentation:** https://github.com/kawastony/Quantum_Gravity
**GitHub Issues:** https://github.com/kawastony/Quantum_Gravity/issues
**Email:** kawastony2@gmail.com or tkawas@scpng.gov.pg

---

**Last Updated:** June 2026
**API Version:** 1.0
**Status:** Production
