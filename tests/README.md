# Tests Directory

This directory contains test cases and testing utilities for the Stellar Cartography Project.

## Purpose

The tests ensure the correctness, reliability, and robustness of the project's code. They serve as executable documentation and help prevent regressions when making changes to the codebase.

## Test Structure

The tests are organized to mirror the structure of the `src` directory:

- **test_data_processing.py** - Tests for data loading and transformation functions
- **test_coordinate_conversion.py** - Tests for coordinate system conversions
- **test_sector_management.py** - Tests for the 3D-MGRS sector implementation
- **test_system_mapping.py** - Tests for system mapping algorithms
- **test_visualization.py** - Tests for visualization functions

## Test Data

The `test_data` subdirectory contains small, representative datasets used specifically for testing purposes. These datasets are designed to exercise the code without requiring the full project dataset.

## Running Tests

Tests can be run using the pytest framework:

```bash
# Run all tests
pytest tests/

# Run specific test file
pytest tests/test_coordinate_conversion.py

# Run with coverage report
pytest --cov=src tests/
```

## Test Writing Guidelines

When adding new tests:

1. Follow test naming conventions: `test_<function_name>_<scenario>`
2. Create small, focused test cases that test a single aspect of functionality
3. Use parametrized tests for testing multiple inputs with similar logic
4. Include edge cases and error conditions
5. Use descriptive assertion messages
6. Keep test data small and self-contained

## Continuous Integration

Tests are automatically run as part of the CI/CD pipeline to ensure code quality on every commit. Failed tests will block merges to the main branch.

## Test Coverage

The goal is to maintain high test coverage (>80%) for critical functionality, especially:
- Coordinate conversion functions
- Sector calculation algorithms
- Data transformation logic 