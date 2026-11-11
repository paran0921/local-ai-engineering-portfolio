# Codex, OpenCode, Ollama, and Laravel: A Windows-Based Local Inference Workflow

**Verification date:** November 11, 2026  
**Status:** Published case study — initial version; follow-up revisions may be made as additional validation is completed.

## 1. Overview

This case study documents a Windows-based development workflow in which Codex provides the development environment, OpenCode runs as a secondary coding-agent CLI, and an Ollama inference server hosted on a separate laptop provides the language model.

The validation focused on resolving execution and configuration problems between the Codex internal terminal, OpenCode, and the remote Ollama endpoint, then verifying that OpenCode could submit a prompt to the configured model and receive the expected response.

The work also included read-only inspection of a Laravel 12 project. It did not include source-code modifications or PHPUnit execution.

## 2. System Architecture

The current architecture uses two machines with distinct responsibilities.

| Machine | Role | Responsibility |
|---|---|---|
| Lenovo ThinkPad X1 Carbon | Development Client | Runs Codex and OpenCode; accesses the Laravel project |
| HP OMEN MAX | Ollama Inference Server | Runs Ollama and provides model inference over the LAN |

The ThinkPad does not currently run its own Ollama model for inference. It acts as the Development Client and sends inference requests to the Ollama Inference Server on the HP OMEN.

Running local inference on a future, higher-specification development laptop is a possible future extension, not part of the configuration validated here.

### Request flow

~~~text
Development Client (ThinkPad)
        |
        | Codex internal terminal
        v
     OpenCode CLI
        |
        | OpenAI-compatible API (/v1)
        v
Ollama Inference Server (HP OMEN)
        |
        v
qwen3:8b-agent-32k
~~~

## 3. Verified Environment

| Component | Verified value |
|---|---|
| Development OS | Windows 11 Pro |
| Inference server OS | Windows 11 Home |
| Codex CLI | 0.166.0 |
| OpenCode | 1.18.33 |
| Ollama | 0.40.16 |
| Laravel project | Laravel 12 |
| PHP requirement | ^8.2 |
| Successful OpenCode model | qwen3:8b-agent-32k |

The Ollama server exposed an OpenAI-compatible API endpoint. The Development Client accessed it over the local network.

The model tag qwen3:8b-agent-32k is the locally configured Ollama model tag. The tag name alone does not establish the active runtime context length; that requires separate runtime verification.

## 4. Problems Encountered and Resolutions

### 4.1 Codex internal terminal could not access the Ollama server

**Observed behavior**

- A TCP connection to the Ollama endpoint succeeded from a separate PowerShell session.
- The same connection attempt failed from the Codex internal terminal.
- Codex status indicated that the Windows sandbox was using elevated sandbox mode and that workspace network access was not enabled.

**Resolution**

After explicit user approval, the Codex user configuration was updated with:

~~~toml
[sandbox_workspace_write]
network_access = true
writable_roots = ['C:\projects\laravel-app']
~~~

Codex was restarted, and the TCP connectivity check from its internal terminal then succeeded.

**Security consideration**

network_access = true enables general outbound network access for the applicable workspace-write sandbox. It is not restricted to the Ollama server or a single destination. This broader permission scope should be considered before adopting the setting in another environment.

A backup of the original configuration was created before the change.

### 4.2 OpenCode initialization and filesystem path errors

OpenCode initially encountered filesystem errors while creating or accessing its default configuration and data directories.

The observed errors included:

- EEXIST while creating the default OpenCode configuration directory.
- FileSystem.open failure when accessing the default OpenCode log path.

Setting only XDG_CONFIG_HOME did not resolve all path-related errors.

**Resolution**

The diagnostic session isolated all four XDG locations under a temporary directory:

~~~powershell
$base = Join-Path $env:TEMP 'opencode-isolated-test'

$env:XDG_CONFIG_HOME = Join-Path $base 'config'
$env:XDG_DATA_HOME   = Join-Path $base 'data'
$env:XDG_STATE_HOME  = Join-Path $base 'state'
$env:XDG_CACHE_HOME  = Join-Path $base 'cache'
~~~

This allowed OpenCode initialization to proceed in the isolated session.

These were session-level diagnostic environment variables, not a permanent system-wide configuration change.

### 4.3 OpenCode did not use the intended project configuration in the diagnostic session

After isolating the XDG paths, OpenCode reported a provider/model resolution error. A model-listing command also returned Provider not found: ollama.

The project-specific OpenCode configuration file existed, but it was not being used in that diagnostic invocation.

**Resolution**

The intended configuration file was explicitly selected for the session. Run this from the Laravel project directory:

~~~powershell
$env:OPENCODE_CONFIG = (Resolve-Path '.oc\config\opencode\opencode.json').Path
~~~

After this variable was set, the following command listed the configured Ollama models:

~~~powershell
opencode models ollama
~~~

The output included:

~~~text
ollama/qwen3:8b
ollama/qwen3:8b-agent-32k
ollama/qwen7-cline:latest
~~~

This confirms that the models were registered and visible to OpenCode in that session. Model registration alone does not prove successful inference.

## 5. Successful Inference Verification

With the isolated XDG directories and explicit project configuration selected, the following command was executed from the Codex internal terminal:

~~~powershell
opencode run --model ollama/qwen3:8b-agent-32k "Reply with exactly: CODEX_OPENCODE_32K_OK"
~~~

Observed result:

~~~text
> build · qwen3:8b-agent-32k
CODEX_OPENCODE_32K_OK
~~~

**Exit code:** 0

This verifies that, in the tested session:

1. The Codex internal terminal could reach the remote Ollama endpoint.
2. OpenCode loaded the explicitly selected project configuration.
3. OpenCode recognized the configured Ollama model.
4. OpenCode submitted a prompt and received the expected model response.

This was a short inference check. It does not establish long-context reliability, tool-calling correctness, autonomous coding capability, or production readiness.

## 6. Read-Only Laravel Project Inspection

The Laravel project was inspected without modifying source files.

### Composer metadata

The composer.json inspection reported:

| Field | Value |
|---|---|
| Package | laravel/laravel |
| PHP requirement | ^8.2 |
| Laravel requirement | ^12.0 |

The Composer metadata command completed successfully.

### User model inspection

The app/Models/User.php source was read and analyzed.

The inspection confirmed:

- Namespace: App\Models
- Traits: HasFactory, Notifiable
- Fillable attributes: name, email, password
- Hidden attributes: password, remember_token
- Casts: email_verified_at as datetime; password as hashed
- Relationship: posts() returns a HasMany relationship
- isAdmin() checks the truthiness of is_admin

The description of isAdmin() is limited to the observed truthiness check. No additional type or authorization semantics are inferred.

## 7. Validation Scope and Limitations

| Validation item | Result |
|---|---|
| Codex internal terminal to Ollama TCP connectivity | Passed |
| Isolated OpenCode initialization | Passed |
| Explicit project configuration selection | Passed |
| Ollama model registration visibility in OpenCode | Passed |
| Short OpenCode inference request | Passed |
| Composer metadata inspection | Passed |
| Laravel User model read-only inspection | Completed |
| PHPUnit test creation | Not performed |
| PHPUnit test execution | Not performed |
| Laravel source modification | Not performed |
| Full agent-driven development workflow | Not verified |
| Long-context behavior and reliability | Not verified |

The successful inference test should therefore be described as a connectivity, configuration, and short-response validation—not as proof that the complete AI-assisted development workflow is ready for production.

## 8. Next Validation Steps

The next work items are separate from the current baseline:

1. Write and execute a PHPUnit test for User::isAdmin().
2. Evaluate OpenCode v2 in a separate environment without replacing the validated OpenCode 1.18.33 baseline.
3. Evaluate Laravel 13 in a separate project or isolated environment.
4. Compare the results and update the documentation only after each validation has supporting evidence.

## 9. Conclusion

The November 11, 2026 validation resolved the observed Codex sandbox network restriction and OpenCode configuration/path issues sufficiently to complete a short inference request through the remote Ollama server.

The verified architecture remains a Development Client on the ThinkPad and an Ollama Inference Server on the HP OMEN.

Laravel inspection was read-only. No Laravel source files were changed, no PHPUnit tests were executed, and no claim is made that autonomous development has been fully validated.

**Publication note:** This is an initial public version. Corrections and additional evidence may be incorporated in subsequent revisions.
