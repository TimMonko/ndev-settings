# ndev-settings Copilot Instructions

## Project Overview

This is a napari plugin that provides a **reusable settings system** for the ndev-kit ecosystem. It uses YAML-based configuration with automatic widget generation for napari UIs. The core architecture follows a **singleton pattern** with dynamic settings loading from multiple sources.

## Key Architecture Patterns

### Singleton Settings Manager

- Use `get_settings()` from `__init__.py` - never instantiate `Settings` directly
- Settings are loaded once and cached globally via `_settings_instance`
- Reset singleton in tests: `ndev_settings._settings_instance = None`

### YAML-First Configuration

```yaml
Group_Name:
  setting_name:
    value: current_value
    default: fallback_value
    tooltip: "User-facing description"
    choices: [option1, option2]  # For dropdowns
    dynamic_choices:             # For runtime-populated dropdowns
      provider: "entry.point.name"
      fallback_message: "No options available"
```

### Entry Points Integration

- External packages contribute settings via `entry_points()`
- Widget choices populated dynamically from installed packages
- External YAML files merge into main settings (preserve main file precedence)

### Testing Strategy

- **Critical**: Use `mock_settings_file_path` fixture in `conftest.py`
- Tests use `test_data/test_settings.yaml` as mock data source
- Always copy test data to `tmp_path` to avoid file corruption
- Mock the `Settings.__init__` method to redirect file paths during tests

## Development Workflows

### Adding New Settings

1. Add to `ndev_settings.yaml` (main file starts empty - see `test_data/test_settings.yaml` for examples)
2. Widget auto-generates in `SettingsContainer`
3. Test with both direct `Settings` instantiation and widget creation

### Widget Development

- Widgets auto-generate from YAML metadata using `magicgui.create_widget()`
- Separate `create_widget_args` from `widget_options` in `_create_widget_for_setting()`
- Use `GroupBoxContainer` for organized UI sections
- Connect changes via `_connect_events()` pattern

### Testing Best Practices

```python
def test_feature(test_settings_file):  # Use fixture, not direct file paths
    settings = Settings(str(test_settings_file))
    # Test logic here
```

## Critical Files

- `_settings.py`: Core Settings class with YAML loading/merging logic
- `_settings_widget.py`: Automatic widget generation for napari
- `conftest.py`: Essential test fixtures and mocking setup
- `test_data/test_settings.yaml`: Complete settings examples for testing
- `napari.yaml`: Napari plugin registration (widget contribution)

## Build & Test Commands

```bash
# Install in development mode
pip install -e .

# Run tests with pytest-qt for widget testing
pytest tests/

# Lint and format (follows Black 79-char limit)
ruff check .
ruff format .
```

## napari Plugin Integration

- Registered via `napari.manifest` entry point in `pyproject.toml`
- Widget command: `ndev-settings.make_settings_container`
- Uses `magicclass` and `magicgui` for UI components
- Settings persist automatically when modified through widgets

## Common Pitfalls

- **Never** instantiate `Settings` directly - always use `get_settings()`
- External YAML files must follow exact same structure as main settings
- Widget creation fails silently if YAML structure is malformed
- Test isolation requires proper singleton reset in fixtures
