# MultiAI Provider Plugin

A comprehensive Unreal Engine plugin that provides a unified interface for interacting with multiple AI providers including Claude, ChatGPT, and Gemini. This plugin simplifies AI integration in your Unreal Engine projects with Blueprint-friendly components and automatic configuration management.

## Features

- **Multi-Provider Support**: Claude (Anthropic), ChatGPT (OpenAI), and Gemini (Google)
- **Blueprint Integration**: Easy-to-use async nodes for Blueprint visual scripting
- **Automatic Configuration**: JSON-based settings persistence with auto-save functionality
- **Asynchronous Operations**: Non-blocking AI conversations with cancellation support
- **Thread-Safe Design**: Safe concurrent access to AI providers
- **Automatic Fallback**: Robust error handling with provider switching capabilities
- **Real-time Monitoring**: Conversation state tracking and progress indicators

## Quick Start

### 1. Installation

1. Copy the `MultiAIProvider` folder to your project's `Plugins` directory
2. Regenerate project files
3. Build your project
4. Enable the plugin in the Unreal Editor

### 2. Basic Setup

#### Adding the Component

Add an `AI Conversation Component` to any Actor in your scene:

```cpp
// C++ Example
UCLASS()
class YOURGAME_API AYourActor : public AActor
{
    GENERATED_BODY()

public:
    AYourActor();

protected:
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "AI")
    class UAIConversationComponent* AIComponent;
};
```

Or add it directly in Blueprint by searching for "AI Conversation Component" in the Components panel.

#### Initial Configuration

The component automatically loads settings on startup. For first-time setup, configure your API keys:

**Blueprint:**
```
1. Get reference to AIConversationComponent
2. Call "Apply Settings" with your configuration
3. Call "Save Settings" to persist the configuration
```

**C++:**
```cpp
FAIProviderSettings Settings;
Settings.CurrentProvider = EAIProviderType::Claude;
Settings.ClaudeAPIKey = TEXT("your-claude-api-key");
Settings.ClaudeModel = TEXT("claude-3-5-haiku-20241022");
Settings.PersonalityPrompt = TEXT("You are a helpful AI assistant.");

AIComponent->ApplySettings(Settings);
AIComponent->SaveSettings();
```

### 3. Basic Usage

#### Sending Conversations

**Blueprint:**
Use the "Send Conversation Async" node:
```
Send Conversation Async -> On Response (handle response)
                       -> On Failed (handle error)
```

**C++:**
```cpp
AIComponent->SendConversation(
    TEXT("Hello, how are you?"),
    [this](const FAIResponse& Response)
    {
        // Handle successful response
        UE_LOG(LogTemp, Log, TEXT("AI Response: %s"), *Response.ResponseText);
    },
    [this](const FAIResponse& ErrorResponse)
    {
        // Handle error
        UE_LOG(LogTemp, Error, TEXT("AI Error: %s"), *ErrorResponse.ErrorMessage);
    }
);
```

#### Loading and Saving Settings

**Blueprint:**
```
Load AI Settings Async -> On Loaded (settings loaded successfully)
                       -> On Failed (load failed, using defaults)

Save AI Settings Async -> On Saved (settings saved)
                       -> On Failed (save failed)
```

**C++:**
```cpp
// Load settings
AIComponent->LoadSettings();

// Save settings
AIComponent->SaveSettings();
```

## Configuration

### Provider Settings Structure

The `FAIProviderSettings` structure contains all configuration options:

```cpp
USTRUCT(BlueprintType)
struct FAIProviderSettings
{
    // Provider Selection
    EAIProviderType CurrentProvider;  // Claude, ChatGPT, or Gemini
    
    // Claude Settings
    FString ClaudeAPIKey;
    FString ClaudeModel;             // e.g., "claude-3-5-haiku-20241022"
    
    // ChatGPT Settings
    FString ChatGPTAPIKey;
    FString ChatGPTModel;            // e.g., "gpt-4o-mini"
    
    // Gemini Settings
    FString GeminiAPIKey;
    FString GeminiModel;             // e.g., "gemini-1.5-flash"
    
    // Common Settings
    FString PersonalityPrompt;       // System prompt for all providers
    int32 MaxTokens;                 // Maximum response length (100-4000)
    float Temperature;               // Response creativity (0.0-2.0)
    bool bAutoSave;                  // Automatically save settings changes
};
```

### Supported Models

#### Claude (Anthropic)
- `claude-3-5-haiku-20241022` (Default - Fast, efficient)
- `claude-3-5-sonnet-20241022` (Balanced performance)
- `claude-3-opus-20240229` (Most capable)

#### ChatGPT (OpenAI)
- `gpt-4o-mini` (Default - Cost-effective)
- `gpt-4o` (Latest GPT-4 model)
- `gpt-4-turbo` (High performance)
- `gpt-3.5-turbo` (Fast, economical)

#### Gemini (Google)
- `gemini-1.5-flash` (Default - Fast responses)
- `gemini-1.5-pro` (Advanced reasoning)
- `gemini-1.0-pro` (Stable, reliable)

### Configuration File

Settings are automatically saved to `ProjectRoot/Config/AIProviderSettings.json`:

```json
{
    "CurrentProvider": "Claude",
    "ClaudeAPIKey": "sk-ant-api03-...",
    "ClaudeModel": "claude-3-5-haiku-20241022",
    "ChatGPTAPIKey": "sk-...",
    "ChatGPTModel": "gpt-4o-mini",
    "GeminiAPIKey": "AIza...",
    "GeminiModel": "gemini-1.5-flash",
    "PersonalityPrompt": "You are a helpful AI assistant.",
    "MaxTokens": 1000,
    "Temperature": 0.7,
    "bAutoSave": true,
    "LastSaved": "2025-08-20T10:30:00Z"
}
```

## Blueprint Nodes

### Async Conversation Node

**Input Pins:**
- `Conversation Component` (AI Conversation Component): Target component
- `Message` (String): Your message to the AI
- `System Prompt` (String): Optional personality/instruction prompt

**Output Pins:**
- `On Response`: Successful AI response received
  - `Response` (AI Response): Contains response text and metadata
- `On Failed`: Error occurred during conversation
  - `Error Response` (AI Response): Contains error details

### Async Settings Nodes

#### Load AI Settings
**Input Pins:**
- `Conversation Component` (AI Conversation Component): Target component

**Output Pins:**
- `On Loaded`: Settings loaded successfully
  - `Loaded Settings` (AI Provider Settings): The loaded configuration
- `On Failed`: Load failed, using defaults
  - `Success` (Boolean): Always false for this pin

#### Save AI Settings
**Input Pins:**
- `Conversation Component` (AI Conversation Component): Target component
- `Settings` (AI Provider Settings): Configuration to save

**Output Pins:**
- `On Saved`: Settings saved successfully
  - `Success` (Boolean): True if saved
- `On Failed`: Save operation failed
  - `Success` (Boolean): False if failed

## Advanced Usage

### Switching Providers at Runtime

```cpp
// Switch to ChatGPT
FAIProviderSettings CurrentSettings = AIComponent->GetCurrentSettings();
CurrentSettings.CurrentProvider = EAIProviderType::ChatGPT;
AIComponent->ApplySettings(CurrentSettings);
```

### Conversation Cancellation

```cpp
// Cancel ongoing conversation
AIComponent->CancelConversation();
```

### Provider Availability Check

```cpp
// Check if Claude is available
bool bClaudeAvailable = AIComponent->IsProviderAvailable(EAIProviderType::Claude);

// Get current provider type
EAIProviderType CurrentType = AIComponent->GetCurrentProviderType();

// Get all available providers
TArray<EAIProviderType> AvailableProviders = AIComponent->GetAvailableProviders();
```

### Event Handling

Bind to component events for real-time updates:

```cpp
// Bind to conversation events
AIComponent->OnConversationStarted.AddDynamic(this, &AYourActor::OnConversationStarted);
AIComponent->OnConversationCompleted.AddDynamic(this, &AYourActor::OnConversationCompleted);
AIComponent->OnConversationFailed.AddDynamic(this, &AYourActor::OnConversationFailed);
AIComponent->OnConversationCancelled.AddDynamic(this, &AYourActor::OnConversationCancelled);

// Bind to settings events
AIComponent->OnSettingsLoaded.AddDynamic(this, &AYourActor::OnSettingsLoaded);
```

## API Key Setup

### Claude (Anthropic)
1. Visit [Anthropic Console](https://console.anthropic.com/)
2. Create an account and navigate to API Keys
3. Generate a new API key (starts with `sk-ant-api03-`)
4. Add the key to your configuration

### ChatGPT (OpenAI)
1. Visit [OpenAI Platform](https://platform.openai.com/)
2. Go to API Keys section
3. Create a new secret key (starts with `sk-`)
4. Add the key to your configuration

### Gemini (Google)
1. Visit [Google AI Studio](https://aistudio.google.com/)
2. Go to API Keys section
3. Create a new API key (starts with `AIza`)
4. Add the key to your configuration

## Error Handling

The plugin provides comprehensive error handling:

### Common Error Types
- **Invalid API Key**: Check your API key configuration
- **Network Error**: Verify internet connection
- **Rate Limit**: API usage limits exceeded
- **Model Not Found**: Verify model name spelling
- **Insufficient Quota**: Check your API account billing

### Error Response Structure
```cpp
struct FAIResponse
{
    FString ResponseText;        // AI response or error message
    bool bSuccess;              // True if successful
    FString ErrorMessage;       // Detailed error description
    int32 ErrorCode;           // HTTP error code (if applicable)
    FEmotionData EmotionData;  // Emotion analysis (if successful)
    FString ProviderName;      // Which provider handled the request
};
```

## Best Practices

### 1. API Key Security
- Never hardcode API keys in your source code
- Use environment variables or secure configuration files
- Consider using different keys for development and production

### 2. Performance Optimization
- Use appropriate models for your use case (faster models for simple tasks)
- Implement conversation caching to reduce API calls
- Set reasonable token limits to control costs

### 3. Error Handling
- Always handle both success and failure cases
- Implement retry logic for transient failures
- Provide fallback options when AI services are unavailable

### 4. User Experience
- Show loading indicators during AI processing
- Allow users to cancel long-running requests
- Provide clear error messages to users

## Troubleshooting

### Common Issues

**Plugin doesn't appear in Editor:**
- Verify plugin is in the correct Plugins folder
- Check that project files were regenerated
- Ensure project compiled successfully

**API calls fail:**
- Verify API keys are correctly formatted
- Check internet connectivity
- Confirm API quotas aren't exceeded

**Settings don't persist:**
- Verify write permissions to Config directory
- Check if auto-save is enabled
- Manually call SaveSettings() after changes

**Conversations hang:**
- Check for network timeouts
- Verify provider availability
- Use CancelConversation() to reset state

### Debug Information

Enable detailed logging by adding to your project's `DefaultEngine.ini`:

```ini
[Core.Log]
LogTemp=Verbose
LogHttp=Verbose
```

Use the debug information method:
```cpp
FString DebugInfo = AIComponent->GetSettingsDebugInfo();
UE_LOG(LogTemp, Warning, TEXT("AI Settings Debug: %s"), *DebugInfo);
```

## License

Copyright 2025 Takashi Marumo. All rights reserved.

## Support

For issues, feature requests, or questions:
1. Check the troubleshooting section above
2. Review the example implementations in the plugin source
3. Consult Unreal Engine documentation for Blueprint usage
4. Verify API provider documentation for service-specific issues

## Version History

### v1.0.0
- Initial release
- Support for Claude, ChatGPT, and Gemini
- Blueprint async nodes
- Automatic configuration management
- Thread-safe operations
- Comprehensive error handling
