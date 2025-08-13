# OpenAI Compatible Provider: `supports_parallel_tool_calls` Configuration

This document explains how to configure the `supports_parallel_tool_calls` parameter for OpenAI compatible language model providers in Zed.

## Background

Some OpenAI-compatible API endpoints don't support the `parallel_tool_calls` parameter and will return an error if it's included in requests. This new configuration option allows you to control whether this parameter is sent to your custom API endpoint.

## Configuration Hierarchy

The `supports_parallel_tool_calls` setting can be configured at two levels:

1. **Provider level**: Applies to all models from that provider (unless overridden)
2. **Model level**: Overrides the provider-level setting for specific models

### Priority Order

When determining whether to include `parallel_tool_calls` in API requests, Zed uses this priority:

1. Model-specific `supports_parallel_tool_calls` setting (highest priority)
2. Provider-level `supports_parallel_tool_calls` setting
3. Default value: `true` (lowest priority)

## Configuration Examples

### Provider-Level Configuration

Set `supports_parallel_tool_calls` at the provider level to apply to all models:

```json
{
  "language_models": {
    "openai_compatible": {
      "my_provider": {
        "api_url": "https://my-api.com/v1",
        "supports_parallel_tool_calls": false,
        "available_models": [
          {
            "name": "model-1",
            "display_name": "Model 1",
            "max_tokens": 4096
          }
        ]
      }
    }
  }
}
```

In this example, all models from `my_provider` will have `parallel_tool_calls` omitted from API requests.

### Model-Level Override

Override the provider setting for specific models:

```json
{
  "language_models": {
    "openai_compatible": {
      "my_provider": {
        "api_url": "https://my-api.com/v1",
        "supports_parallel_tool_calls": false,
        "available_models": [
          {
            "name": "basic-model",
            "display_name": "Basic Model",
            "max_tokens": 4096
          },
          {
            "name": "advanced-model",
            "display_name": "Advanced Model (supports parallel tools)",
            "max_tokens": 8192,
            "supports_parallel_tool_calls": true
          }
        ]
      }
    }
  }
}
```

In this example:
- `basic-model` inherits the provider setting (`false`)
- `advanced-model` overrides it with `true`

### Mixed Configuration

You can have different providers with different default behaviors:

```json
{
  "language_models": {
    "openai_compatible": {
      "legacy_provider": {
        "api_url": "https://legacy-api.com/v1",
        "supports_parallel_tool_calls": false,
        "available_models": [
          {
            "name": "legacy-model",
            "display_name": "Legacy Model",
            "max_tokens": 2048
          }
        ]
      },
      "modern_provider": {
        "api_url": "https://modern-api.com/v1",
        "available_models": [
          {
            "name": "modern-model",
            "display_name": "Modern Model",
            "max_tokens": 8192
          }
        ]
      }
    }
  }
}
```

In this example:
- `legacy_provider` explicitly disables parallel tool calls
- `modern_provider` uses the default (`true`) since no setting is specified

## API Behavior

When `supports_parallel_tool_calls` is:
- `true`: The `parallel_tool_calls: false` parameter is included in API requests when tools are present
- `false`: The `parallel_tool_calls` parameter is completely omitted from API requests

## Troubleshooting

### Error: "Unknown parameter: parallel_tool_calls"

If you see this error from your API endpoint, set `supports_parallel_tool_calls: false` for that provider or specific model.

### Tools not working as expected

If tool calling isn't working with your custom provider, try:
1. Ensure your API supports the OpenAI tools format
2. Check if `supports_parallel_tool_calls: false` resolves any parameter-related errors
3. Verify your API endpoint URL is correct

## Default Behavior

If you don't specify `supports_parallel_tool_calls` anywhere in your configuration, Zed will default to `true` and include `parallel_tool_calls: false` in requests (when tools are present), maintaining backward compatibility with the standard OpenAI API format.