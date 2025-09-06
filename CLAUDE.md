# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cubes is a Python OLAP (Online Analytical Processing) framework for multidimensional data analysis. It provides a logical abstraction layer over physical data stores, focusing on how analysts think about data rather than physical implementation details.

## Development Commands

### Installation and Setup
```bash
# Install package in development mode with all dependencies
pip install -e .[dev]

# Install with specific components
pip install -e .[sql]     # SQL backend only
pip install -e .[slicer]  # Server components
pip install -e .[all]     # All components
```

### Running Tests
```bash
# Run all tests
python -m unittest discover tests

# Run specific test module
python -m unittest tests.test_model
python -m unittest tests.sql.test_browser

# The test suite is defined in setup.py as "tests"
python setup.py test
```

### Documentation
```bash
# Build documentation (from doc/ directory)
cd doc
make html       # Build HTML documentation
make clean      # Clean build artifacts
```

### Running the Server
```bash
# Run the OLAP server (requires config file)
slicer serve slicer.ini

# With custom visualizer URL
slicer serve slicer.ini --visualizer http://example.com

# Debug mode
CUBES_DEBUG=1 slicer serve slicer.ini
```

## Architecture Overview

### Core Module Structure

**`cubes/metadata/`** - Model Definition Layer
- `model.py` - Core model classes (Cube, Dimension, Hierarchy, Level, Attribute)
- `dimension.py` - Dimension-specific logic and flat dimensions
- `cube.py` - Cube metadata and measure aggregates
- `providers.py` - Model loading from various sources (files, bundles)
- Key concept: Logical models abstract physical database structure

**`cubes/query/`** - Query and Browsing Engine
- `browser.py` - Abstract browser interface for data aggregation
- `cells.py` - Cell definitions (points and ranges in multidimensional space)
- `computation.py` - Calculated measures and post-aggregation computations
- Key concept: Browsers translate logical queries into backend-specific operations

**`cubes/sql/`** - SQL Backend Implementation
- `browser.py` - SQL-specific browser implementing ROLAP operations
- `mapper.py` - Maps logical model to physical SQL tables/columns
- `query.py` - SQL query builder for aggregations
- `store.py` - SQL data store connection management
- Key concept: Generates optimized SQL for star/snowflake schemas

**`cubes/server/`** - HTTP API Server
- `blueprint.py` - Flask blueprint defining API endpoints
- `browser.py` - Handles cube browsing requests
- `auth.py` - Authentication and authorization
- `store.py` - Store management for multiple data sources
- Key concept: RESTful API for OLAP operations

**`cubes/workspace.py`** - Configuration and Coordination
- Central configuration management
- Model and store initialization
- Browser factory for different backends
- Namespace management for multi-tenant setups

### Key Design Patterns

1. **Logical vs Physical Separation**: The metadata layer defines logical business concepts while mappers translate to physical implementation
2. **Backend Abstraction**: Query browser interface allows multiple backend implementations (SQL, MongoDB, etc.)
3. **Hierarchical Dimensions**: Supports drill-down/roll-up through dimension hierarchies (e.g., year→month→day)
4. **Localization**: Model metadata supports multiple languages through localizable attributes
5. **Expression-based Attributes**: Computed attributes using the `expressions` library for derived measures

### Model Configuration

Models are defined in JSON/YAML with:
- **Cubes**: Facts with measures and dimension links
- **Dimensions**: Hierarchical attributes for slicing data
- **Aggregates**: Sum, count, min, max, avg operations on measures
- **Mappings**: Physical table/column references

Example configuration typically in `slicer.ini`:
```ini
[workspace]
model = model.json
stores = stores.ini

[server]
host = 0.0.0.0
port = 5000
```

### Extension System

Cubes uses an extension system (`cubes/ext.py`) for:
- Model providers (file, bundle, mixpanel)
- Store backends (sql, mongo, slicer)
- Authentication methods
- Formatters (json, csv, xlsx)

Extensions are discovered via entry points or explicit registration.