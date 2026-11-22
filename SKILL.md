# Solana hard push

Hard-push a Solana game from recon to a running driver: map the game, log in, play it around the clock on the free lane, and report only what matters. Proven by Dreadwood, SolHero, Tarvion, Pokedex Idle, Dracoria, and Antropolis in November 2026.

## Recon before playing

- Identify the game first: URL, chain, login type, free entry cost, game token and its mint address, and the withdrawal path (minimum, fee, review).

- Read the client JavaScript and the terms before touching the game. Check for a one-account rule, an automation ban, a withdrawal minimum, and any paid funnel behind the free lane.

- Report the entry cost and the ban risk before playing. A "grind, push, speedrun" order never authorizes spending. Free play only until the user explicitly approves a purchase.

- Look for profitable sybil loopholes and flag the ban risk of each one, so the user decides with eyes open. One account plus a sanctioned in-game agent beats a banned sybil farm at payout review.

## Login is usually SIWS

- Most Solana games log in with Sign-In With Solana: fetch a challenge, sign the message bytes with ed25519, post the signature, receive a session cookie.

- The Binance Agentic Wallet cannot sign Solana messages. It signs EIP-712 typed data only. A SIWS login needs either a transient in-memory signature from the recovery phrase (never printed, never stored, one-shot script deleted after) or the user logging in himself.

- Store the session in a chmod 600 file under the game directory. Never print the phrase, the key, or the session token. Note the session expiry when it is known.

- Use a browser user agent on API calls. A non-browser agent can get a literal guest or bot response.

- A watchdog re-enables an in-game agent or relaunches a driver, but it never re-logs in. Re-login is the parent agent's job.

## Play API-first, browser last

- Map the same-origin API endpoints from the client bundles and play through them. API calls are faster than browser automation and survive VM restarts better.

- Use the browser only when the flow needs page JavaScript, such as wallet injection or a canvas game. The default browser is Brave on the VM driven by Playwright, never the cloud browser, which cannot execute page JavaScript at all.

- Before concluding a game system does not exist, refresh the client code from the live site and search every client file, including gate and loader scripts. A stale local copy plus a partial search missed the entire Dreadwood events system in November 2026.

- Check the state a snapshot was written at before trusting it. A bag dump written once at boot stays stale all day. Read the live state instead.

## Driver plus supervisor plus watchdog

- Run the game as three layers. The driver is one detached Node process with a tick loop that writes state.json and a log. The supervisor relaunches the driver when it dies. The watchdog cron checks liveness every 10 minutes and reports only milestones, sales, or a ban.

- Expect the driver to die about every 10 minutes and on every VM restart. Verify it is alive by PID plus working directory plus a fresh lastTick before trusting any of its numbers.

- Keep masters under ~/workspace/<game>/. Never keep them in /tmp, which VM restarts wipe. Global npm installs vanish on redeploy too, while node_modules under ~/workspace survive.

- Kill a driver by its exact PID after checking /proc/PID/cwd. Several games run a driver.mjs and a driver_supervisor.sh, and PIDs alone do not say which game they belong to. Some drivers ignore SIGTERM, so SIGKILL plus a supervisor relaunch is the working restart for those.

## Solana mechanics that matter

- Token-2022 mints need TOKEN_2022_PROGRAM_ID, not the classic token program id.

- Confirm a Solana transfer by polling getSignatureStatuses over HTTP, never by websocket.

- Solana RPC works direct from the VM. Game sites are direct-first with the chain relay at 127.0.0.1:18080 as fallback. Market reads can be slow, so budget generous timeouts.

- Wallet balance and in-game balance are separate numbers. Never mix them in a report. A deposit is verified twice: once from the on-chain receipt, once from the in-game credit.

- Read the live game code for drop and reward rules instead of trusting memory. Mining found zero gems in Dreadwood because gemDrops was false in the live build, and only raids and dungeons paid gems.

## Money rules

- Never buy a game token, swap proceeds onward, or pay for a mint without the user's explicit yes for that exact action.

- Report sales and milestones only. Routine progress stays silent. Never re-report an unchanged persistent blocker.

- Check the withdrawal minimum, fee, and review process before promising any cash-out. A thin market and a 10 percent spread turn a big token balance into a small payout.

## Worked examples

- Dreadwood (play.dreadwood.xyz): idle castle town on Solana. Driver plus supervisor plus a 10-minute reporter. Keep 5 landed 2026-11-22, then Goblin Warren dungeon runs for 8 gems a day. In-game $DREAD is separate from the wallet balance. Three tome sales at 20,000 $DREAD each, net 19,000 after the 5 percent tax.

- SolHero (play.playsolhero.com): idle anime RPG. Automation is explicitly banned by the game, so the ban risk was flagged and the user ordered play anyway, eyes open. Driver ticks every 90 seconds, watchdog relaunches it, reports only gear sales. Hero Bebingah the Archer reached Lv 68.

- Pokedex Idle (play.pokedex.fund): free Pokemon MMO. SIWS login done transiently, then the game's own built-in Agent does the farming on safe defaults (no diamond spend, no market buys, no release or trade). Auto-list script sells box duplicates at the 10 PokeGems floor, gated by a hidden birth-score rule of 3.5 or higher.

- Dracoria: Solana airdrop farming across 100 batch wallets plus 5 egg accounts on a 12-hour loop. All dragons to L4, egg buy at balance 75,000 or higher, PVP only on fee-0 low and medium risk targets. Balances vanished twice with no confirmed cause, so every run reconciles the total.

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.

- If nothing here fits, look on the web for a maintained API or MCP server and set it up with [Custom Connectors and MCPs](/opt/muse/skills/custom-connector-setup/SKILL.md), or build a custom connector.

## Alternatives for better results

- When a game ships a sanctioned in-game agent or auto-player, use it instead of outside automation. It is the only route that cannot be banned for botting.

- When the API shape is unknown and the game is canvas-only, Brave plus Playwright with UI clicks is the fallback. Budget human pacing and expect the session to need the user's own device when logins block the VM.
