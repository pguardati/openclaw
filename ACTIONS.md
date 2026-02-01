# Installation and Testing Actions Log

## Session: 2026-02-01

### Completed Actions

1. **Checked prerequisites**
   - Node version: 24.8.0 (required: >=22)
   - pnpm version: 9.4.0

2. **Installed dependencies**
   - Ran `pnpm install`
   - 954 packages installed (911 downloaded, 43 from cache)
   - Native modules compiled: sharp, node-pty, esbuild, node-llama-cpp, matrix-sdk-crypto-nodejs

3. **Built the project**
   - Ran `pnpm build`
   - TypeScript compiled successfully
   - Canvas A2UI bundled

4. **Ran linting**
   - `pnpm lint` - 0 errors, 0 warnings

5. **Ran tests**
   - `pnpm test` - 208 tests passed across 34 test files

6. **Tested CLI**
   - `pnpm openclaw --help` - working

7. **Ran doctor**
   - Fixed permissions on ~/.openclaw (chmod 700)
   - Fixed permissions on ~/.openclaw/openclaw.json (chmod 600)
   - Created missing directories: agents/main/sessions, credentials

8. **Configured gateway mode**
   - Set `gateway.mode` to `local`

9. **Started dev gateway**
   - Generated auth token for dev profile
   - Started gateway on port 19001

10. **Verified gateway status**
    - Gateway reachable at ws://127.0.0.1:19001
    - Dashboard at http://127.0.0.1:19001/

11. **Configured Ollama**
    - Ollama is running with 2 models: qwen3-coder:latest, deepseek-coder-v2:16b
    - Both models have tool capability
    - Added explicit provider config to ~/.openclaw-dev/openclaw.json
    - Config synced to ~/.openclaw-dev/agents/main/agent/models.json
    - Models appear in `models list --all` with Local=yes, Auth=yes

12. **Set default model**
    - Set `ollama/qwen3-coder:latest` as default model
    - Command: `pnpm openclaw --dev models set ollama/qwen3-coder:latest`

13. **Tested AI agent with Ollama**
    - Command: `pnpm openclaw --dev agent --message "Hello, what model are you?" --agent dev --local`
    - Agent responded using local Qwen3-Coder model
    - Working successfully

14. **TUI debugging**
    - TUI showed "connected | idle" but no output
    - Issue: gateway.remote.token was not set
    - Fixed by setting `gateway.remote.token` to match `gateway.auth.token`
    - After restart, previous answers appeared (confirms messages were processed but not streamed)

15. **TUI streaming issue**
    - TUI connects but doesn't display streamed output (possible bug)
    - Agent runs complete successfully in background
    - **Workaround: Use CLI instead of TUI**

16. **Web UI works**
    - URL: `http://127.0.0.1:19001/?token=dev-token-123`
    - Chat works correctly with Ollama model
    - TUI and CLI have issues, but Web UI is functional

### Summary
OpenClaw installed and working with local Ollama (qwen3-coder:latest).
- Gateway: ws://127.0.0.1:19001
- Dashboard: http://127.0.0.1:19001/
- TUI: `pnpm openclaw --dev tui`
- CLI: `pnpm openclaw --dev agent --message "..." --agent dev --local`

### Quick Reference Commands

**Start services (2 terminals):**
```bash
# Terminal 1: Ollama
ollama serve

# Terminal 2: Gateway
pnpm openclaw --dev gateway --verbose --token "dev-token-123"
```

**Open Web UI (works):**
```
http://127.0.0.1:19001/?token=dev-token-123
```

**Stop services:**
- Each terminal: `Ctrl+C`
- Kill all: `pkill -f "openclaw.*gateway" ; pkill -f ollama`

### Chech Core Capabilities                                                                                                       
                                                                                                                          
1. Multi-Channel Messaging                                                                                              
Connect your AI to messaging platforms you already use:                                                                 
- WhatsApp, Telegram, Slack, Discord, Signal, iMessage                                                                  
- Google Chat, MS Teams, Matrix, and more (13+ channels)                                                                
                                                                                                                        
openclaw channels add telegram                                                                                          
openclaw channels add whatsapp                                                                                          
openclaw channels status                                                                                                
                                                                                                                        
2. Voice Interaction                                                                                                    
- Voice Wake - hands-free "Hey OpenClaw" activation (macOS/iOS/Android)                                                 
- Talk Mode - continuous conversation overlay                                                                           
                                                                                                                        
3. Browser Control                                                                                                      
Let the agent browse the web, take screenshots, and automate tasks:                                                     
openclaw config set browser.enabled true                                                                                
                                                                                                                        
4. Canvas & Visual UI                                                                                                   
Agent-controlled visual workspace for rendering interactive content, charts, and custom UIs.                            
                                                                                                                        
5. Automation                                                                                                           
- Cron jobs - schedule recurring tasks                                                                                  
- Webhooks - trigger actions from external services                                                                     
- Gmail integration - email-based triggers                                                                              
                                                                                                                        
6. Device Nodes                                                                                                         
Pair iOS/Android devices as remote nodes with camera, screen recording, and canvas access.                              
                                                                                                                        
7. Skills                                                                                                               
Extend capabilities with installable skill packages:                                                                    
openclaw skills list                                                                                                    
openclaw skills install <skill-name>                                                                                    
                                                                                                                        
Quick Next Steps                                                                                                        
                                                                                                                        
1. Add a messaging channel to chat from your phone:                                                                     
openclaw channels add whatsapp                                                                                          
2. Try the agent directly:                                                                                              
openclaw agent --message "What can you help me with?"                                                                   
3. Check status:                                                                                                        
openclaw status --all                                                                                                   
4. Explore skills:                                                                                                      
openclaw skills list --available                 

17. **Configured Discord channel**
    - Ran `pnpm openclaw channels add` (interactive, don't pass channel name as argument)
    - Selected Discord, entered bot token
    - Set DM policy to pairing (recommended)
    - Named account: pguardati-bot

18. **Gateway token configuration**
    - **Important:** `--token` flag doesn't work reliably
    - Must set token via config first:
      ```bash
      pnpm openclaw --dev config set gateway.auth.token "dev-token-123"
      ```
    - Then start gateway:
      ```bash
      pkill -9 -f "openclaw-gateway"
      pnpm openclaw --dev gateway run --verbose
      ```

19. **Fixed Discord config profile mismatch**
    - **Problem:** `openclaw channels add` (without `--dev`) saves config to `~/.openclaw/openclaw.json`
    - But gateway running with `--dev` uses `~/.openclaw-dev/openclaw.json`
    - Discord token was in wrong profile → bot showed "access not configured"
    - **Fix:** Always use `--dev` flag consistently:
      ```bash
      pnpm openclaw --dev channels add
      ```
    - Or manually copy Discord config (token, name, dm settings) to dev profile
    - **Pairing workaround:** Pairing approve was buggy, used allowlist instead:
      ```bash
      pnpm openclaw --dev config set channels.discord.dm.policy "allowlist"
      pnpm openclaw --dev config set 'channels.discord.dm.allowFrom' '["YOUR_DISCORD_USER_ID"]'
      ```
    - Discord user ID: 1409935562810331287

### Key Lesson: Profile Consistency
- `--dev` flag uses separate config directory (`~/.openclaw-dev/`)
- Without `--dev` uses main config (`~/.openclaw/`)
- **Always use `--dev` consistently for all commands when running dev gateway**

20. **Enabled browser tool**
    - Command: `pnpm openclaw --dev config set browser.enabled true`
    - Browser tool supports: start, navigate, snapshot, screenshot, act (click, type, etc.)

21. **Tested browser automation via Discord**
    - Asked agent to navigate and click a button
    - **Issue:** Local Ollama model used `web_fetch` instead of `browser` tool
    - Local models struggle with proper tool selection
    - Need to be very explicit: "Use the browser tool with action=navigate..."

22. **Attempted OpenAI configuration (Vocareum)**
    - Added OpenAI provider with custom base URL:
      ```json
      "openai": {
        "baseUrl": "https://openai.vocareum.com/v1",
        "apiKey": "voc-...",
        "models": [{"id": "gpt-4o", ...}]
      }
      ```
    - Set default model to `openai/gpt-4o`
    - **Result:** 401 Incorrect API key error
    - Vocareum API rejected the key - may need different auth method

23. **Reverted to Ollama**
    - Switched default model back to `ollama/qwen3-coder:latest`
    - OpenAI config remains in file but not used as default

### Running Processes (check before shutdown)
```bash
ps aux | grep -E "openclaw|ollama" | grep -v grep
```
- `ollama serve` (PID varies) - local model server
- `openclaw-gateway` (PID varies) - OpenClaw gateway

### Stop Commands
```bash
pkill -9 -f "openclaw-gateway"  # Stop gateway
pkill -f "ollama serve"          # Stop Ollama (optional)
```

### Key Lessons Learned
1. **Profile consistency:** Always use `--dev` flag for all commands when running dev gateway
2. **Config via CLI:** Some flags like `--token` don't work; use `config set` instead
3. **Browser tool:** Local models may not select correct tools; need explicit instructions
4. **API compatibility:** Custom OpenAI endpoints may need different auth formats

### Next Steps
- Get valid OpenAI/Anthropic API key for better tool use
- Explore skills and plugins
- Try other messaging channels (Telegram, WhatsApp)