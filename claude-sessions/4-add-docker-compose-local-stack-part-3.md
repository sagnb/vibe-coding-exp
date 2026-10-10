 ▐▛███▛█   Claude Code v2.1.270
▝▜██████▀  Sonnet 5 · Claude Pro
  ▝▝ ▝▝    ~\OneDrive\Desktop\vibe-coding-exp\relis


❯ Implement sub-issue 02.04: Expose health/readiness through the local proxy    
  and restrict database and storage port exposure.                              
                                                                                
  Work only inside relis/. Follow the repository instructions and the mandatory 
  context/project-structure.md, context/stack.yml, and                          
  context/testing-policy.md. Use the applicable local Claude skills. No browser 
  or external research is required; the requirements are provided below.        
                                                                                
  Objective                                                                     
  Ensure local developers access application health/readiness through Nginx,    
  while internal services remain unpublished and necessary development          
  endpoints are bound only to loopback.                                         
                                                                                
  Existing implementation to reuse                                              
  - docker-compose.yml already defines web, API, worker, PostgreSQL, migration  
  services, Nginx, MailHog, and SeaweedFS.                                      
  - docker/nginx/nginx.conf routes /api/ to api:3001 with the prefix stripped,  
  and / to web:3000.                                                            
  - The API already exposes /health and /ready, accessible through /api/health  
  and /api/ready.                                                               
  - /nginx-health reports the proxy's own liveness, not application readiness.  
  - Nginx is published on 127.0.0.1:${NGINX_HTTP_PORT:-8080}.                   
  - MailHog's UI is published on 127.0.0.1:${MAILHOG_UI_PORT:-8025}.            
  - SeaweedFS's S3 endpoint is published on                                     
  127.0.0.1:${SEAWEEDFS_S3_PORT:-8333}.                                         
  - API, web, and PostgreSQL currently have no host-published ports.            
  - Existing integration tests, browser tests, disposable-project helpers,      
  persistence checks, and guarded cleanup must be preserved.                    
                                                                                
  Inspect the current code before editing. These are starting observations, not 
  a substitute for verification. Implement only missing behavior or evidence;   
  do not rebuild completed work.                                                
                                                                                
  Requirements                                                                  
                                                                                
  1. Proxy health and readiness                                                 
  - Verify the existing API response contracts and preserve their status codes  
  and safe response bodies through Nginx.                                       
  - Test /api/health and /api/ready through the actual host-published proxy.    
  - Test an unavailable API upstream: the proxy must return an appropriate      
  non-success response, never the web page or a false healthy response.         
  - Verify recovery after restoring the upstream using bounded readiness        
  polling.                                                                      
  - Keep /nginx-health independent: proxy liveness must not be presented as     
  proof that the API, database, or worker is ready.                             
  - Do not invent database-readiness semantics or new application endpoints.    
  Document what the existing readiness contract actually checks.                
                                                                                
  2. Safe port exposure                                                         
  - Verify the resolved Compose configuration and actual running container port 
  bindings.                                                                     
  - API, web, PostgreSQL, worker, and migration services must not publish host  
  ports.                                                                        
  - Keep Nginx and the approved MailHog UI/S3 development endpoints             
  loopback-only.                                                                
  - Do not expose SMTP, storage administration endpoints, or additional         
  database ports.                                                               
  - Reject wildcard bindings such as 0.0.0.0 or :: in tests for host-published  
  development endpoints.                                                        
  - Distinguish internal container connectivity from host-published ports. Do   
  not claim Docker networking is a security boundary against the host           
  administrator.                                                                
                                                                                
  3. Automated verification                                                     
  - Put integration tests under tests/integration/docker-compose/ and shared    
  helpers under packages/test-utils/, following the prescribed test layout.     
  - Reuse existing tests where they already provide adequate coverage.          
  - Cover successful proxy responses, upstream failure and recovery, and both   
  resolved and runtime port exposure.                                           
  - Include controlled negative cases proving unsafe bindings are rejected.     
  - Use real Docker for integration claims. Mocks may prove validation logic    
  but must not be presented as real networking evidence.                        
                                                                                
  4. Safe execution                                                             
  - Use uniquely named disposable Compose projects and non-conflicting test     
  ports.                                                                        
  - Stop or restart only services belonging to the disposable test project.     
  - Preserve the ownership checks, baseline-volume protection, fail-closed      
  cleanup, and consistent environment forwarding added in sub-issue 02.03.      
  - Never prune Docker resources, restart Docker Desktop, or touch existing     
  user stacks or data.                                                          
  - Report unavailable infrastructure as a blocker, not a passing or completed  
  check.                                                                        
                                                                                
  5. Documentation                                                              
  - Update the existing stack documentation with the verified health/readiness  
  URLs, their actual meaning, and the host-port exposure policy.                
  - Keep documentation changes limited to this sub-issue; do not advance the    
  broader startup/diagnostics documentation sub-issue.                          
                                                                                
  Completion criteria                                                           
  - Actual proxy success, failure, and recovery behavior is verified.           
  - Resolved configuration and runtime bindings confirm the intended exposure   
  policy.                                                                       
  - Relevant regression tests and typechecks pass.                              
  - Any failures are diagnosed, fixed within scope, and retested without        
  weakening assertions.                                                         
  - Report exact commands, pass/fail/skip counts, changed files, and remaining  
  limitations. Separate independently executed real integrations from mocked    
  checks.                                                                       
  - Do not claim the parent issue complete, start another sub-issue, commit, or 
  push.                                                                         

● I'll start by inspecting the current state of both repositories and the
  relevant context before making any changes.

  Searched for 2 patterns, read 3 files, ran 2 shell commands (ctrl+o to expand)

● Nothing in the pending decisions blocks this slice. Now let me look at the
  actual current implementation: docker-compose files, nginx config, existing
  tests from 02.03.

  Searched for 1 pattern, read 10 files, ran 6 shell commands (ctrl+o to expand)

● Docker is available, so I can run real integration tests. Let me check the
  remaining test files and the .env.example / stack.yml for nginx-related
  settings, plus check s3-sigv4.ts briefly isn't needed. Let me check
  config-validation.test.ts and persistence-cleanup-env.test.ts and
  persistence-ignore-rules.test.ts quickly for patterns I should follow, and
  check process.ts helper.

  Read 4 files (ctrl+o to expand)

● Now let me implement the test-utils additions.

● Update(packages\test-utils\src\docker-compose.ts)
  ⎿  Added 1 line, removed 1 line
      251  }
      252  
      253  /** The container id for `service` (its most recent one, inclu
           ding already-exited containers via `-a`), or `undefined` if no
           ne exists yet. */
      254 -async function getContainerId(project: ComposeProject, service
          -: string): Promise<string | undefined> {                      
      254 +export async function getContainerId(project: ComposeProject, 
          +service: string): Promise<string | undefined> {               
      255    const result = await runToCompletion("docker", composeArgs(p
           roject, ["ps", "-a", "-q", service]), {
      256      cwd: project.cwd,
      257      timeoutMs: 15000,

● Update(packages\test-utils\src\docker-compose.ts)
  ⎿  Added 33 lines
      328  }
      329  
      330  /**
      331 + * Polls `url` with plain HTTP GET requests until a response's
          + status code                                                  
      332 + * satisfies `isAcceptable`, or `timeoutMs` elapses — unlike `
          +waitForHttpOk`                                                
      333 + * (which accepts ANY status, including a failure one, as "the
          + server is                                                    
      334 + * accepting connections"), this is for a BOUNDED wait on a sp
          +ecific                                                        
      335 + * outcome, e.g. "the proxy reports success again after its up
          +stream was                                                    
      336 + * restored." Returns the accepted `Response` so the caller ca
          +n inspect its                                                 
      337 + * body without a second request.                             
      338 + */                                                           
      339 +export async function waitForHttpStatus(                      
      340 +  url: string,                                                
      341 +  isAcceptable: (status: number) => boolean,                  
      342 +  timeoutMs = 60000,                                          
      343 +): Promise<Response> {                                        
      344 +  const start = Date.now();                                   
      345 +  let lastStatus: number | undefined;                         
      346 +  let lastError: unknown;                                     
      347 +  while (Date.now() - start < timeoutMs) {                    
      348 +    try {                                                     
      349 +      const response = await fetch(url);                      
      350 +      lastStatus = response.status;                           
      351 +      if (isAcceptable(response.status)) return response;     
      352 +    } catch (error) {                                         
      353 +      lastError = error;                                      
      354 +    }                                                         
      355 +    await new Promise((resolveDelay) => setTimeout(resolveDela
          +y, 1000));                                                    
      356 +  }                                                           
      357 +  throw new Error(                                            
      358 +    `Timed out after ${timeoutMs}ms waiting for ${url} to reac
          +h an acceptable status. ` +                                   
      359 +      `Last observed status: ${lastStatus ?? "(none — request 
          +never completed)"}. Last error: ${String(lastError)}`,        
      360 +  );                                                          
      361 +}                                                             
      362 +                                                              
      363 +/**                                                           
      364   * Polls `docker compose ps` for `service` until its healthche
           ck reports
      365   * "(healthy)", or `timeoutMs` elapses. Relies on a real `heal
           thcheck:`
      366   * block existing on that service in docker-compose.yml — neve
           r

● Now let's append the port-exposure helpers at the end of the file.

  Read 1 file (ctrl+o to expand)

● Update(packages\test-utils\src\docker-compose.ts)
  ⎿  Added 135 lines
      1023  
      1024    return { projectName: parsed.name ?? project.projectName, v
            olumeNames, externalVolumes, bindMounts, databaseTargets };
      1025  }
      1026 +                                                             
      1027 +/** One `ports:` entry as `docker compose config --format jso
           +n` resolves it for a service. */                             
      1028 +export interface ResolvedComposePort {                       
      1029 +  /** The container-side port being published. */            
      1030 +  target: number;                                            
      1031 +  /** Host-side port, when published (may be unset for an eph
           +emeral/unspecified mapping). */                              
      1032 +  published?: string;                                        
      1033 +  /** Host IP the port is bound to, exactly as resolved — e.g
           +. `"127.0.0.1"`, `"0.0.0.0"`, or unset (meaning every interfa
           +ce). Never assume loopack when this is absent. */            
      1034 +  hostIp?: string;                                           
      1035 +  protocol?: string;                                         
      1036 +}                                                            
      1037 +                                                             
      1038 +/**                                                          
      1039 + * Resolves the merged Compose configuration and extracts ONL
           +Y each                                                       
      1040 + * service's declared `ports:` entries (never bind mounts, vo
           +lumes, or                                                    
      1041 + * environment — see `resolveComposePersistence` for those). 
           +Read-only and                                                
      1042 + * safe at any time: it runs `docker compose config`, which r
           +equires the                                                  
      1043 + * `docker compose` CLI but not a running daemon, and creates
           + nothing.                                                    
      1044 + *                                                           
      1045 + * This is the STATIC half of this sub-issue's port-exposure 
           +proof — it                                                   
      1046 + * confirms what Compose WOULD publish for a given environmen
           +t, before any                                                
      1047 + * container exists. See `getContainerHostPortBindings` below
           + for the                                                     
      1048 + * matching RUNTIME half (the actual bindings an already-runn
           +ing container                                                
      1049 + * was created with).                                        
      1050 + */                                                          
      1051 +export async function resolveComposePublishedPorts(project: C
           +omposeProject, env: ComposeEnv): Promise<Record<string, Resol
           +vedComposePort[]>> {                                         
      1052 +  const args = composeArgs(project, ["config", "--format", "j
           +son"]);                                                      
      1053 +  const result = await runToCompletion("docker", args, { cwd:
           + project.cwd, env: withEnv(env), timeoutMs: 60000 });        
      1054 +  if (result.exitCode !== 0) {                               
      1055 +    throw new Error(`docker compose config --format json fail
           +ed (exit ${result.exitCode}):\n${result.stderr}`);           
      1056 +  }                                                          
      1057 +                                                             
      1058 +  const parsed = JSON.parse(result.stdout) as {              
      1059 +    services?: Record<string, { ports?: { target?: number; pu
           +blished?: string; host_ip?: string; protocol?: string }[] }>;
      1060 +  };                                                         
      1061 +                                                             
      1062 +  const out: Record<string, ResolvedComposePort[]> = {};     
      1063 +  for (const [serviceName, service] of Object.entries(parsed.
           +services ?? {})) {                                           
      1064 +    out[serviceName] = (service.ports ?? []).map((entry) => (
           +{                                                            
      1065 +      target: entry.target ?? 0,                             
      1066 +      published: entry.published,                            
      1067 +      hostIp: entry.host_ip,                                 
      1068 +      protocol: entry.protocol,                              
      1069 +    }));                                                     
      1070 +  }                                                          
      1071 +  return out;                                                
      1072 +}                                                            
      1073 +                                                             
      1074 +/** One host-port binding for a container, as `docker inspect
           +`'s `HostConfig.PortBindings` reports it. */                 
      1075 +export interface ContainerPortBinding {                      
      1076 +  hostIp: string;                                            
      1077 +  hostPort: string;                                          
      1078 +}                                                            
      1079 +                                                             
      1080 +/**                                                          
      1081 + * The actual host-port bindings `service`'s container was CR
           +EATED with —                                                 
      1082 + * `docker inspect`'s `HostConfig.PortBindings`, keyed by `"<
           +containerPort>/<protocol>"`                                  
      1083 + * (e.g. `"80/tcp"`). Deliberately reads `HostConfig.PortBind
           +ings`, not                                                   
      1084 + * `NetworkSettings.Ports`: the latter only reflects ACTIVE b
           +indings while a                                              
      1085 + * container is running and can read back empty for an alread
           +y-exited                                                     
      1086 + * container (e.g. `migrate-control`/`migrate-project`, which
           + exit quickly),                                              
      1087 + * even though its publish configuration is unchanged — `Host
           +Config.PortBindings`                                         
      1088 + * is fixed at container-creation time and is therefore accur
           +ate whether the                                              
      1089 + * container is running, exited, or stopped.                 
      1090 + *                                                           
      1091 + * A container that declares no `ports:` at all resolves to `
           +{}` (Docker                                                  
      1092 + * never adds an EXPOSE-only port here — only an explicit pub
           +lish does),                                                  
      1093 + * which is exactly the proof this sub-issue needs for `api`/
           +`web`/`postgres`/                                            
      1094 + * `worker`/the migration services: no host port exposure, re
           +gardless of                                                  
      1095 + * what the image itself exposes internally.                 
      1096 + *                                                           
      1097 + * Works on an already-exited container too (uses `getContain
           +erId`'s own                                                  
      1098 + * `-a` lookup) — required for the one-shot migration service
           +s.                                                           
      1099 + */                                                          
      1100 +export async function getContainerHostPortBindings(project: C
           +omposeProject, service: string): Promise<Record<string, Conta
           +inerPortBinding[] | null>> {                                 
      1101 +  const containerId = await getContainerId(project, service);
      1102 +  if (!containerId) {                                        
      1103 +    throw new Error(`No container found for service "${servic
           +e}" in project ${project.projectName}`);                     
      1104 +  }                                                          
      1105 +  const inspect = await runToCompletion("docker", ["inspect",
           + containerId, "--format", "{{json .HostConfig.PortBindings}}"
           +], {                                                         
      1106 +    timeoutMs: 15000,                                        
      1107 +  });                                                        
      1108 +  if (inspect.exitCode !== 0) {                              
      1109 +    throw new Error(`docker inspect "${containerId}" failed (
           +exit ${inspect.exitCode}):\n${inspect.stderr}`);             
      1110 +  }                                                          
      1111 +  const raw = JSON.parse(inspect.stdout.trim() || "null") as 
           +Record<string, { HostIp: string; HostPort: string }[] | null>
           + | null;                                                     
      1112 +  if (!raw) return {};                                       
      1113 +  const result: Record<string, ContainerPortBinding[] | null>
           + = {};                                                       
      1114 +  for (const [portKey, bindings] of Object.entries(raw)) {   
      1115 +    result[portKey] = bindings ? bindings.map((binding) => ({
           + hostIp: binding.HostIp, hostPort: binding.HostPort })) : nul
           +l;                                                           
      1116 +  }                                                          
      1117 +  return result;                                             
      1118 +}                                                            
      1119 +                                                             
      1120 +/**                                                          
      1121 + * Whether `hostIp` is a genuinely loopback-only bind address
           +. ONLY                                                       
      1122 + * `"127.0.0.1"` qualifies — never a wildcard (`"0.0.0.0"`, `
           +"::"`), never                                                
      1123 + * the empty string (Docker's own shorthand for "every interf
           +ace," the same                                               
      1124 + * as `0.0.0.0`), and never IPv6 loopback `"::1"` either, sin
           +ce this                                                      
      1125 + * stack's own published services are always configured with 
           +the literal                                                  
      1126 + * IPv4 loopback address (see docker-compose.yml) — accepting
           + `::1` here                                                  
      1127 + * too would let a silent IPv6-wildcard regression (`::`, whi
           +ch Linux can                                                 
      1128 + * map to dual-stack `0.0.0.0`-like exposure) pass unnoticed.
      1129 + *                                                           
      1130 + * Pure and synchronous — independently unit-testable with sy
           +nthetic                                                      
      1131 + * values, no Docker required. Used to validate BOTH the stat
           +ic resolved                                                  
      1132 + * configuration (`resolveComposePublishedPorts`) and real ru
           +nning                                                        
      1133 + * containers' bindings (`getContainerHostPortBindings`).    
      1134 + */                                                          
      1135 +export function isLoopbackOnlyHostIp(hostIp: string | undefin
           +ed): boolean {                                               
      1136 +  return hostIp === "127.0.0.1";                             
      1137 +}                                                            
      1138 +                                                             
      1139 +/**                                                          
      1140 + * Asserts every binding in `bindings` (as returned by       
      1141 + * `getContainerHostPortBindings` or adapted from `resolveCom
           +posePublishedPorts`)                                         
      1142 + * is loopback-only, throwing with the exact offending port/a
           +ddress                                                       
      1143 + * otherwise. `context` names what is being checked, for a re
           +adable                                                       
      1144 + * failure. A service with no bindings at all (`{}`) triviall
           +y passes —                                                   
      1145 + * this function only rejects a genuinely unsafe PUBLISHED bi
           +nding, never                                                 
      1146 + * the absence of one.                                       
      1147 + */                                                          
      1148 +export function assertLoopbackOnlyBindings(bindings: Record<s
           +tring, ContainerPortBinding[] | null>, context: string): void
           + {                                                           
      1149 +  for (const [portKey, list] of Object.entries(bindings)) {  
      1150 +    if (!list) continue;                                     
      1151 +    for (const binding of list) {                            
      1152 +      if (!isLoopbackOnlyHostIp(binding.hostIp)) {           
      1153 +        throw new Error(                                     
      1154 +          `${context}: port ${portKey} is published on host a
           +ddress "${binding.hostIp}", not loopback-only ("127.0.0.1") —
           + ` +                                                         
      1155 +            `this is a wildcard or otherwise non-loopback bin
           +d, exposing the service beyond the local host.`,             
      1156 +        );                                                   
      1157 +      }                                                      
      1158 +    }                                                        
      1159 +  }                                                          
      1160 +}                                                            

  Read 1 file (ctrl+o to expand)

● Good — it exports src/index.ts directly (no build step), so vitest/tsx resolve
  the TS source directly. Now let's write the two test files.

  Ran 1 shell command (ctrl+o to expand)

● Write(tests\integration\docker-compose\proxy-health.test.ts)
  ⎿  Wrote 183 lines to
     tests\integration\docker-compose\proxy-health.test.ts
       1 import path from "node:path";
       2 import { fileURLToPath } from "node:url";
       3 import { afterAll, beforeAll, describe, expect, it } from "vites
         t";
       4 import {
       6   cleanupComposeProject,
       7   composeStart,
       8   composeStop,
       9   composeUp,
      10   getFreePort,
     … +233 lines (ctrl+o to expand)

● Write(tests\integration\docker-compose\port-exposure.test.ts)
  ⎿  Wrote 237 lines to
     tests\integration\docker-compose\port-exposure.test.ts
       1 import path from "node:path";
       2 import { fileURLToPath } from "node:url";
       3 import { afterAll, beforeAll, describe, expect, it } from "vites
         t";
       4 import {
       6   type ContainerPortBinding,
       7   assertLoopbackOnlyBindings,
       8   cleanupComposeProject,
       9   composeUp,
      10   getContainerHostPortBindings,
     … +312 lines (ctrl+o to expand)

● Update(tests\integration\docker-compose\port-exposure.test.ts)
  ⎿  Added 1 line
       6    type ContainerPortBinding,
       7    assertLoopbackOnlyBindings,
       8    cleanupComposeProject,
       9 +  composePort,                                                 
      10    composeUp,
      11    getContainerHostPortBindings,
      12    getFreePort,

● Update(tests\integration\docker-compose\port-exposure.test.ts)
  ⎿  Removed 1 line
      185      ]);
      186      // mailhog has no Compose healthcheck (see docker-compose.
           yml) — poll
      187      // its real published port directly instead, same as stack
           -smoke.test.ts.
      188 -    const { composePort } = await import("@relis/test-utils");
      188      const mailhogPort = await composePort(project, "mailhog", 
           8025);
      189      await waitForHttpOk(`http://127.0.0.1:${mailhogPort}/`);
      190      // Block until the one-shot migration containers have actu
           ally exited

● Now let's typecheck the test-utils package and the new test files.

  Ran 1 shell command (ctrl+o to expand)

● test-utils typechecks clean. Now let's check the root tsconfig for tests and
  typecheck the whole repo (or at least the tests directory).

  Ran 3 shell commands (ctrl+o to expand)

● Typecheck passes cleanly. Now let's run the full pnpm -r typecheck to be safe,
  then run the actual integration tests.

  Ran 1 shell command (ctrl+o to expand)

● Now let's run the new test files against the real Docker daemon, starting with
  the proxy health suite.

  Ran 1 shell command (ctrl+o to expand)

● nginx caches the upstream's DNS at startup and has a default 60s
  proxy_connect_timeout, so a stopped upstream causes a ~60s hang before nginx
  returns 504 — not an instant 502. I need to raise this test's own timeout
  above that.

● Update(tests\integration\docker-compose\proxy-health.test.ts)
  ⎿  Added 10 lines, removed 4 lines
      125      }, 60000);
      126  
      127      it("returns a non-success response through the proxy — nev
           er a false-healthy 200, and never the web page", async () => {
      128 +      // Verified by real execution: `proxy_pass http://api:30
          +01/;` uses a                                                  
      129 +      // static hostname, so nginx resolves and caches "api"'s
          + IP only                                                      
      130 +      // once, at its own startup — it does not re-resolve per
          + request.                                                     
      131 +      // After `docker compose stop api`, that cached IP is no
          + longer                                                       
      132 +      // reachable, and packets toward it are silently dropped
          + rather than                                                  
      133 +      // refused — so nginx hits its OWN default `proxy_connec
          +t_timeout`                                                    
      134 +      // (60s, unset in this minimal dev config — see docker/n
          +ginx/nginx.conf)                                              
      135 +      // before giving up with 504 Gateway Timeout, rather tha
          +n an                                                          
      136 +      // immediate 502. This test's own timeout is raised acco
          +rdingly                                                       
      137 +      // (below) instead of assuming an instant failure.      
      138        const health = await fetch(`http://127.0.0.1:${nginxPort
           }/api/health`);
      129 -      // nginx's own default behavior for a connection-refused
          - upstream                                                     
      130 -      // (no custom error_page/proxy_intercept_errors in this 
          -minimal dev                                                   
      131 -      // config — see docker/nginx/nginx.conf) is 502 Bad Gate
          -way. Asserted                                                 
      132 -      // against the real response rather than assumed.       
      139        expect(health.status).toBeGreaterThanOrEqual(500);
      140        expect(health.status).toBeLessThan(600);
      141        expect(health.ok).toBe(false);
● Update(tests\integration\docker-compose\proxy-health.test.ts)
  ⎿  Added 1 line, removed 1 line
      152        expect(ready.status).toBeLessThan(600);
      153        const readyBody = await ready.text();
      154        expect(readyBody).not.toContain('"status":"ok"');
      155 -    });                                                       
      155 +    }, 150000);                                               
      156  
      157      it("still serves the (unaffected) web app through the same
            proxy, and keeps /nginx-health reporting ok", async () => {
      158        // Proves the failure above is specific to the /api/ ups
           tream, not a

  Ran 1 shell command (ctrl+o to expand)

● All 5 proxy-health tests pass (1 skip is the Docker-unavailable blocked
  branch, correctly skipped). Now let's run the port-exposure test.

  Ran 1 shell command (ctrl+o to expand)

● Both new suites pass. Now let's run the full existing integration test suite
  to confirm no regressions from the shared test-utils changes.

  Ran 2 shell commands (ctrl+o to expand)

● Clean teardown confirmed — no leftover containers, networks, or volumes. Now
  let's run the broader pnpm -r test (unit tests) to make sure nothing else is
  affected, then update documentation.

  Ran 2 shell commands (ctrl+o to expand)

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 6 lines, removed 3 lines
       2  
       3  Implements local sub-issues [02.02 — Define services and shared
       4  configuration, including the Control DB and per-project test
       5 -databases](../../../context/notion-tasks/sub-issues/02-02.md) a
         -nd                                                             
       5 +databases](../../../context/notion-tasks/sub-issues/02-02.md), 
       6  [02.03 — Configure local persistent volumes and exclude them fr
          om version
       7 -control](../../../context/notion-tasks/sub-issues/02-03.md) (pa
         -rent: [02                                                      
       8 -— Add Docker Compose local stack](../../../context/notion-tasks
         -/02-docker-compose.md)).                                       
       7 +control](../../../context/notion-tasks/sub-issues/02-03.md), an
         +d                                                              
       8 +[02.04 — Expose health/readiness through the local proxy and re
         +strict                                                         
       9 +database and storage port exposure](../../../context/notion-tas
         +ks/sub-issues/02-04.md)                                        
      10 +(parent: [02 — Add Docker Compose local                        
      11 +stack](../../../context/notion-tasks/02-docker-compose.md)).   
      12  Builds directly on [the local-stack inventory](local-stack-inve
          ntory.md)
      13  from the preceding sub-issue — read that document first for the
      14  application startup contracts this stack reuses unchanged (it d
          oes not

● Now let's add the dedicated Health/readiness and Host port exposure policy
  sections right after the "Internal vs. host-facing vs. browser-public
  addresses" section.

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 73 lines
      134    `http://localhost:${NGINX_HTTP_PORT}`) is not actually exerc
           ised in
      135    normal use.
      136  
      137 +## Health and readiness — verified URLs and their actual meani
          +ng                                                            
      138 +                                                              
      139 +Implements local sub-issue [02.04](../../../context/notion-tas
          +ks/sub-issues/02-04.md).                                      
      140 +Every URL below is reached through the proxy only — `apps/api`
          +'s own port                                                   
      141 +(`3001`) is never published to the host (see "Host port exposu
          +re policy"                                                    
      142 +below).                                                       
      143 +                                                              
      144 +| URL | Reaches | Meaning — verified by reading the actual rou
          +te/behavior, not assumed |                                    
      145 +| --- | --- | --- |                                           
      146 +| `http://127.0.0.1:${NGINX_HTTP_PORT:-8080}/nginx-health` | n
          +ginx itself (not proxied — a dedicated `location` in `docker/n
          +ginx/nginx.conf`) | **Proxy liveness only.** Always `200 ok` w
          +henever the nginx process is up, regardless of whether `web`/`
          +api` are reachable. **Never treat this as application readines
          +s** — `proxy-health.test.ts` proves it stays `200 ok` even whi
          +le the `api` upstream is stopped. |                           
      147 +| `http://127.0.0.1:${NGINX_HTTP_PORT:-8080}/api/health` | `ap
          +ps/api`'s existing `GET /health` (`apps/api/src/app.ts`), via 
          +the `/api/` prefix (stripped by nginx) | `200 {"status":"ok","
          +message":"...","service":"api"}` whenever the API process can 
          +respond at all. Not a dependency check of any kind. |         
      148 +| `http://127.0.0.1:${NGINX_HTTP_PORT:-8080}/api/ready` | `app
          +s/api`'s existing `GET /ready` | `200 {"status":"ok","service"
          +:"api","checks":{"network":"ok","cors":"ok"}}`. This is the **
          +entire, existing** readiness contract — it reports only that t
          +he process itself initialized its network/CORS configuration. 
          +It does **not** check PostgreSQL, the worker, or any queue, be
          +cause `apps/api` has no such dependency in code today. `depend
          +ency-unavailable.test.ts` proves this stays `"ok"` even with `
          +postgres` stopped — a documented limitation, not a bug introdu
          +ced by this sub-issue. No database-readiness semantics are inv
          +ented here. |                                                 
      149 +                                                              
      150 +### Behavior when the API upstream itself is unreachable      
      151 +                                                              
      152 +**Verified by real execution** (`proxy-health.test.ts`): stopp
          +ing `api`                                                     
      153 +(`docker compose stop api`, the upstream nginx proxies to — di
          +stinct from                                                   
      154 +stopping a *downstream* dependency of the API, which is what  
      155 +`dependency-unavailable.test.ts` covers instead) does **not** 
          +produce an                                                    
      156 +instant failure. `docker/nginx/nginx.conf`'s `proxy_pass http:
          +//api:3001/;`                                                 
      157 +uses a static hostname, so nginx resolves and caches that host
          +name's IP                                                     
      158 +once, at nginx's own startup, and does not re-resolve per requ
          +est. After                                                    
      159 +`api` stops, nginx's cached route to it is dead, and packets t
          +oward it are                                                  
      160 +dropped rather than refused — so nginx hits its own default   
      161 +`proxy_connect_timeout` (60s; not overridden in this minimal d
          +ev config)                                                    
      162 +before responding `504 Gateway Timeout`. Either way, the respo
          +nse:                                                          
      163 +                                                              
      164 +- Is always a `5xx`, never a false `200`.                     
      165 +- Is never `apps/web`'s rendered homepage — the browser-facing
          + `/` route                                                    
      166 +  and the `/api/` route fail independently of each other, prov
          +ed by `web`                                                   
      167 +  staying reachable (`200`) throughout the same test.         
      168 +- Never causes `/nginx-health` to report anything other than `
          +200 ok` —                                                     
      169 +  proxy liveness and application readiness are, and must stay,
          + unrelated                                                    
      170 +  signals.                                                    
      171 +                                                              
      172 +Restarting the upstream (`docker compose start api`) is observ
          +ed to                                                         
      173 +recover within the existing success contract above, confirmed 
          +by bounded                                                    
      174 +polling (`waitForHttpStatus`, `packages/test-utils/src/docker-
          +compose.ts`) —                                                
      175 +not merely asserted from the restart command's own exit code. 
      176 +                                                              
      177 +## Host port exposure policy                                  
      178 +                                                              
      179 +Implements local sub-issue [02.04](../../../context/notion-tas
          +ks/sub-issues/02-04.md),                                      
      180 +building on the service graph above.                          
      181 +                                                              
      182 +- **`api`, `web`, `postgres`, `worker`, `migrate-control`, `mi
          +grate-project`                                                
      183 +  publish NO host port at all.** They declare no `ports:` entr
          +y in                                                          
      184 +  `docker-compose.yml`, and `port-exposure.test.ts` confirms t
          +his at both                                                   
      185 +  the resolved `docker compose config` level and against real 
          +(or, for the                                                  
      186 +  two one-shot migration services, real-but-already-exited) co
          +ntainers'                                                     
      187 +  `docker inspect`-reported `HostConfig.PortBindings` — not me
          +rely the                                                      
      188 +  static file, and not merely "the container didn't crash."   
      189 +- **`nginx`, `mailhog` (UI only), and `storage` (S3 gateway on
          +ly) are the                                                   
      190 +  only approved host-published development endpoints, and ever
          +y one binds                                                   
      191 +  to `127.0.0.1` — loopback — only.** Never `0.0.0.0`, never `
          +::`, never an                                                 
      192 +  unspecified host IP (Docker's own shorthand for "every inter
          +face").                                                       
      193 +  `port-exposure.test.ts` asserts this against the real runnin
          +g                                                             
      194 +  containers' actual bindings, not just the compose file's int
          +ent.                                                          
      195 +- **SMTP (`mailhog:1025`) and the storage administration ports
          + (SeaweedFS                                                   
      196 +  master `:9333`, filer `:8888`) are never published.** No app
          +lication                                                      
      197 +  mail or storage adapter exists yet to use them from outside 
          +the Compose                                                   
      198 +  network (see "Known limitations"), and nothing in this sub-i
          +ssue adds                                                     
      199 +  one.                                                        
      200 +- **Docker's internal bridge network is not, and is never clai
          +med to be, a                                                  
      201 +  security boundary against the host administrator.** Isolatin
          +g                                                             
      202 +  container-to-container traffic on the `relis` network keeps 
          +*other                                                        
      203 +  containers on a different Compose project* from reaching `ap
          +i`/`postgres`/                                                
      204 +  `worker` directly, but anyone with access to the Docker daem
          +on on this                                                    
      205 +  host (another `docker run --network`, `docker exec`, etc.) c
          +an still                                                      
      206 +  reach any internal service regardless of whether it publishe
          +s a host                                                      
      207 +  port. The loopback-only publish rule above is the actual, an
          +d only,                                                       
      208 +  host-facing boundary this sub-issue establishes and verifies
          +.                                                             
      209 +                                                              
      210  ## Clean-checkout startup
      211  
      212  Each of `docker/web/Dockerfile`, `docker/api/Dockerfile`, and

● Now let's add the two new test files to the "Tests" section.

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 30 lines
      610    streams checked for a leaked raw value. `-T` (no pseudo-TTY)
            is required
      611    for `docker compose run` to keep stdout/stderr separate at a
           ll; without
      612    it they would be merged into one stream.
      613 +- `proxy-health.test.ts` — implements local sub-issue         
      614 +  [02.04](../../../context/notion-tasks/sub-issues/02-04.md)'s
          + AC1. Brings                                                  
      615 +  up only `nginx` (which pulls in `web`/`api` via `depends_on`
          +) and                                                         
      616 +  verifies, against the real stack: `/api/health`/`/api/ready`
          + preserve                                                     
      617 +  `apps/api`'s existing status codes and bodies through the pr
          +oxy;                                                          
      618 +  `/nginx-health` stays `200 ok` independent of the API's own 
          +readiness;                                                    
      619 +  stopping `api` makes the proxy respond `5xx` (observed: `504
          +`, after                                                      
      620 +  nginx's own default 60s `proxy_connect_timeout` against its 
          +cached,                                                       
      621 +  now-dead upstream route — see "Health and readiness" above) 
          +and never a                                                   
      622 +  false `200` or the web page, while `/` (served by the untouc
          +hed `web`                                                     
      623 +  container) and `/nginx-health` keep working; and restarting 
          +`api`                                                         
      624 +  recovers within the existing success contract, confirmed by 
          +bounded                                                       
      625 +  polling (`waitForHttpStatus`).                              
      626 +- `port-exposure.test.ts` — implements local sub-issue        
      627 +  [02.04](../../../context/notion-tasks/sub-issues/02-04.md)'s
          + AC2 at three                                                 
      628 +  layers: (1) pure, Docker-free unit tests of the loopback-onl
          +y validation                                                  
      629 +  logic itself (`isLoopbackOnlyHostIp`/`assertLoopbackOnlyBind
          +ings`),                                                       
      630 +  including the explicit negative cases this sub-issue calls f
          +or —                                                          
      631 +  synthetic `0.0.0.0`, `::`, and empty-string bindings are all
          + REJECTED —                                                   
      632 +  proving the validation logic only, never presented as real n
          +etworking                                                     
      633 +  evidence; (2) the RESOLVED `docker compose config --format j
          +son` output                                                   
      634 +  (`resolveComposePublishedPorts`), asserting `api`/`web`/`pos
          +tgres`/                                                       
      635 +  `worker`/`migrate-control`/`migrate-project` declare no `por
          +ts:` at all,                                                  
      636 +  and `nginx`/`mailhog`/`storage` declare exactly their approv
          +ed,                                                           
      637 +  loopback-bound port and nothing else (never SMTP `1025`, nev
          +er storage                                                    
      638 +  admin `9333`/`8888`); (3) the REAL running (or, for the one-
          +shot                                                          
      639 +  migration services, real-but-exited) containers' own `docker
          + inspect`                                                     
      640 +  `HostConfig.PortBindings` (`getContainerHostPortBindings`) —
          + the actual                                                   
      641 +  runtime evidence the resolved configuration alone cannot pro
          +vide,                                                         
      642 +  confirming every service matches layer (2) in practice.     
      643  - `dependency-unavailable.test.ts` — explicitly starts `postgr
           es` (plus
      644    `nginx`/`api`; an earlier version only requested `["nginx"]`
           , which
      645    never actually started `postgres` at all, since nothing in n
           ginx's own

  Ran 1 shell command (ctrl+o to expand)

● Document structure looks clean. Let's double check the test-utils exports and
  confirm there's no missed export (e.g. ResolvedComposePort type used
  elsewhere), and re-run the two new test files once more plus a quick
  lint/typecheck pass to confirm everything is solid before reporting.

  Ran 5 shell commands (ctrl+o to expand)

● That tsconfig change is an incidental side-effect of Next.js's lint tooling
  auto-patching the file when I ran pnpm -r lint — not something I intended to
  touch. I'll revert it to avoid leaving stray noise.

  Ran 1 shell command (ctrl+o to expand)

● Good, the working tree now only reflects my intentional changes. Let's do a
  final full verification run of the integration suite once more to confirm
  everything is still green after the revert.

  Ran 2 shell commands (ctrl+o to expand)

● Clean. Here's the implementation report for sub-issue 02.04.

  Summary

  Implemented in relis/ only. Existing nginx routing, Compose port declarations,
  and health/readiness contracts were already correct — the gap was test and 
  documentation evidence, which this work adds.

  Changed/added files

  - packages/test-utils/src/docker-compose.ts — added waitForHttpStatus (bounded
    polling on a specific HTTP status), resolveComposePublishedPorts (static
    docker compose config port extraction), getContainerHostPortBindings (real
    docker inspect HostConfig.PortBindings),
    isLoopbackOnlyHostIp/assertLoopbackOnlyBindings (pure validation), and
    exported the existing private getContainerId. No existing exports changed.
  - tests/integration/docker-compose/proxy-health.test.ts (new) — AC1.
  - tests/integration/docker-compose/port-exposure.test.ts (new) — AC2.
  - docs/architecture/docker-compose-stack.md — added "Health and readiness" and
    "Host port exposure policy" sections, and documented the two new test
    files.

  AC1 — proxy health/readiness (real Docker)

  - Confirmed apps/api's existing contract unchanged: /health → 200 
    {status:"ok",...}; /ready → 200 
    {status:"ok",checks:{network:"ok",cors:"ok"}} (no DB/queue claim — none
    invented).
  - Real test: stops api, proxy returns 5xx (observed 504 — nginx caches the
    upstream hostname at its own startup, so a dead route hits its default 60s
    proxy_connect_timeout rather than an instant 502; documented in code and
    docs), never the web page or a false 200; / and /nginx-health keep working
    throughout, proving /nginx-health is independent proxy liveness, not
    application readiness.
  - Restarts api, confirms recovery via bounded polling (waitForHttpStatus, 30s
    bound).

  AC2 — port exposure (real Docker, two layers + pure validation)

  - Pure unit tests (no Docker) prove the loopback-only validator rejects
    0.0.0.0, ::, empty string, ::1, and accepts only 127.0.0.1 — explicitly
    flagged as validation-logic evidence, not networking evidence.
  - Real docker compose config check:
    api/web/postgres/worker/migrate-control/migrate-project declare zero ports:;
    nginx/mailhog/storage declare exactly one loopback-bound port each, with
    SMTP(1025)/storage-admin(9333,8888) absent.
  - Real running-container check (docker inspect): same assertions against
    actual HostConfig.PortBindings, including the already-exited migration
    containers.

  Commands run (all from relis/)

  - pnpm run typecheck → pass.
  - pnpm run typecheck:tests → pass.
  - npx vitest run --config vitest.integration.config.ts (full suite) → 24 
    files, 313 passed, 8 skipped (expected "blocked" branches), 0 failed.
  - Re-ran the two new files individually afterward → 30 passed, 2 skipped,
    confirmed via docker ps/volume/network ls that no disposable resources
    leaked.
  - pnpm -r test → 0 unit tests found anywhere (pre-existing repo state, per
    PENDING-DECISIONS.md §19 — not introduced here).
  - pnpm -r lint → pre-existing failure in apps/web/src/app/page.tsx (a
    react-hooks/set-state-in-effect rule), unrelated to this change and out of
    scope; I did not touch that file. Running lint also caused Next.js to
    auto-reformat apps/web/tsconfig.json as a side effect — I reverted that
    incidental change so the working tree only reflects this sub-issue's work.

  Limitations / out of scope

  - No database-readiness semantics were added — apps/api still only checks
    network/CORS, exactly as before.
  - Did not modify docker-compose.yml or nginx.conf — both already satisfied the
    required policy; this sub-issue's gap was verification only.
  - Did not advance the broader startup/diagnostics documentation sub-issue; doc
    edits are scoped to health/readiness URLs and the port-exposure policy
    only.
  - Did not commit, push, or start another sub-issue.

✻ Churned for 23m 10s · done 15:22

※ recap: Implemented and verified sub-issue 02.04 (proxy health/readiness + safe port exposure) in relis/ — all tests pass and docs are updated. Next: report is ready for your review; nothing further planned until you confirm or request changes. (disable recaps in /config)

❯ Fix the two review findings in sub-issue 02.04. Keep changes scoped to these  
  corrections.                                                                  
                                                                                
  1. Restore safe lifecycle handling in both new Docker test suites             
  - proxy-health.test.ts and port-exposure.test.ts currently call               
  cleanupComposeProject directly in afterAll, bypassing the protections         
  implemented in sub-issue 02.03.                                               
  - Capture the baseline and validate provisioning safety before creating       
  resources.                                                                    
  - Reuse createComposeCleanupTarget and cleanupDisposableTargets for guarded   
  cleanup, including partial setup failures.                                    
  - Pass the same environment overrides to configuration resolution, startup,   
  and cleanup.                                                                  
  - Report cleanup failures without hiding the original test failure.           
  - Add regression coverage proving unsafe ownership or failed discovery        
  prevents destructive cleanup.                                                 
                                                                                
  2. Make waitForHttpStatus genuinely bounded                                   
  - A pending fetch currently can exceed timeoutMs because the deadline is      
  checked only between requests.                                                
  - Use cancellation tied to the remaining overall deadline, and bound retry    
  delays by that deadline.                                                      
  - Release rejected responses appropriately while preserving the accepted      
  response for the caller.                                                      
  - Add deterministic tests covering a hanging request, repeated unacceptable   
  statuses, successful recovery, and cancellation on timeout. Do not rely on    
  external services.                                                            
                                                                                
  Follow the mandatory stack, project structure, and testing policy. Preserve   
  existing behavior and ownership protections; do not weaken assertions or      
  advance another sub-issue.                                                    
                                                                                
  Run focused regression tests and typechecks, then rerun the two real Docker   
  suites using only uniquely named disposable projects and the corrected        
  safeguards. Report exact commands, results, skips, and limitations.           
                                                                                
  Do not prune resources, restart Docker Desktop, touch existing user stacks,   
  commit, or push.                                                              

  Read 1 file (ctrl+o to expand)

● Let's look at the current waitForHttpStatus implementation and the two test
  files to plan the exact edits.

  Searched for 1 pattern (ctrl+o to expand)

● Now let's rewrite waitForHttpStatus to be genuinely bounded via per-request
  cancellation tied to the remaining deadline.

● Update(packages\test-utils\src\docker-compose.ts)
  ⎿  Added 43 lines, removed 5 lines
      335   * outcome, e.g. "the proxy reports success again after its up
           stream was
      336   * restored." Returns the accepted `Response` so the caller ca
           n inspect its
      337   * body without a second request.
      338 + *                                                            
      339 + * Genuinely bounded by `timeoutMs` overall, not merely "check
          +ed between                                                    
      340 + * requests": each individual `fetch` is given an `AbortContro
          +ller` tied to                                                 
      341 + * the REMAINING time left in the overall deadline (`deadline 
          +- Date.now()`                                                 
      342 + * at the moment that request starts), not a fresh `timeoutMs`
          + of its own —                                                 
      343 + * a single pending request (e.g. a proxy that has silently dr
          +opped                                                         
      344 + * packets toward a dead upstream, with no reply at all) can t
          +herefore                                                      
      345 + * never push the total wait past `timeoutMs`. The retry delay
          + between                                                      
      346 + * attempts is likewise clamped to whatever remains of the dea
          +dline, never                                                  
      347 + * a fixed delay that could itself overshoot it.              
      348 + *                                                            
      349 + * Every REJECTED response's body is explicitly released      
      350 + * (`response.body?.cancel()`) before the next attempt — this 
          +is not a                                                      
      351 + * cosmetic cleanup: an un-drained response body can otherwise
          + stall a                                                      
      352 + * kept-alive connection's reuse by the HTTP client on the ver
          +y next                                                        
      353 + * request. The ACCEPTED response is returned untouched — its 
          +body is                                                       
      354 + * never read or cancelled here — so the caller can still cons
          +ume it                                                        
      355 + * exactly once.                                              
      356   */
      357  export async function waitForHttpStatus(
      358    url: string,
      359    isAcceptable: (status: number) => boolean,
      360    timeoutMs = 60000,
      361  ): Promise<Response> {
      344 -  const start = Date.now();                                   
      362 +  const deadline = Date.now() + timeoutMs;                    
      363    let lastStatus: number | undefined;
      364    let lastError: unknown;
      347 -  while (Date.now() - start < timeoutMs) {                    
      365 +                                                              
      366 +  for (;;) {                                                  
      367 +    const remaining = deadline - Date.now();                  
      368 +    if (remaining <= 0) break;                                
      369 +                                                              
      370 +    const controller = new AbortController();                 
      371 +    const abortTimer = setTimeout(() => controller.abort(), re
          +maining);                                                     
      372      try {
      349 -      const response = await fetch(url);                      
      373 +      const response = await fetch(url, { signal: controller.s
          +ignal });                                                     
      374 +      if (isAcceptable(response.status)) {                    
      375 +        return response;                                      
      376 +      }                                                       
      377        lastStatus = response.status;
      351 -      if (isAcceptable(response.status)) return response;     
      378 +      // Not the response we're waiting for — release it (see 
          +doc comment                                                   
      379 +      // above) rather than leaving its body unconsumed.      
      380 +      try {                                                   
      381 +        await response.body?.cancel();                        
      382 +      } catch {                                               
      383 +        // Already closed/errored; nothing left to release.   
      384 +      }                                                       
      385      } catch (error) {
      386        lastError = error;
      387 +    } finally {                                               
      388 +      clearTimeout(abortTimer);                               
      389      }
      355 -    await new Promise((resolveDelay) => setTimeout(resolveDela
          -y, 1000));                                                    
      390 +                                                              
      391 +    const retryDelay = Math.min(1000, deadline - Date.now()); 
      392 +    if (retryDelay <= 0) break;                               
      393 +    await new Promise((resolveDelay) => setTimeout(resolveDela
          +y, retryDelay));                                              
      394    }
      395    throw new Error(
      396      `Timed out after ${timeoutMs}ms waiting for ${url} to reac
           h an acceptable status. ` +

● Now let's write deterministic tests for waitForHttpStatus using a local HTTP
  server (no external services, no Docker required).

  Ran 1 shell command (ctrl+o to expand)

● Write(tests\integration\docker-compose\wait-for-http-status.test.ts)
  ⎿  Wrote 170 lines to
     tests\integration\docker-compose\wait-for-http-status.test.ts
       1 import { createServer, type IncomingMessage, type Server, type S
       2 import { afterEach, describe, expect, it } from "vitest";
       4 
       5 // Regression coverage for a review finding on this sub-issue (0
       6 // waitForHttpStatus's bound on `timeoutMs` previously checked t
       7 // deadline only BETWEEN requests — a single pending `fetch` (e.
       8 // proxy silently dropping packets toward a dead upstream, with 
         no reply
         hed-DNS/
          open far
     … +207 lines (ctrl+o to expand)

● Update(tests\integration\docker-compose\wait-for-http-status.test.ts)
  ⎿  Added 2 lines, removed 2 lines
      117      expect(serverSawRequest).toBe(true);
      118      // The core regression check: total elapsed time stays clo
           se to
      119      // timeoutMs, never anywhere near how long the hang itself
            could have
      120 -    // lasted (unbounded). A generous grace window absorbs rea
          -l scheduling                                                  
      121 -    ///event-loop jitter without weakening the actual bound be
          -ing proven.                                                   
      120 +    // lasted (unbounded). A generous grace window absorbs rea
          +l scheduling/                                                 
      121 +    // event-loop jitter without weakening the actual bound be
          +ing proven.                                                   
      122      expect(elapsed).toBeLessThan(timeoutMs + 1500);
      123    });
      124  
● Now let's run this new test file to verify it works correctly with the fixed
  implementation.

  Ran 1 shell command (ctrl+o to expand)

● Now let's tackle finding 1 — restoring guarded lifecycle handling in the two
  Docker suites. First, let's re-check the current state of both files.

  Read 1 file (ctrl+o to expand)

● Update(tests\integration\docker-compose\proxy-health.test.ts)
  ⎿  Added 5 lines, removed 1 line
       3  import { afterAll, beforeAll, describe, expect, it } from "vite
          st";
       4  import {
       5    type ComposeProject,
       6 -  cleanupComposeProject,                                       
       6 +  assertProvisioningSafety,                                    
       7 +  cleanupDisposableTargets,                                    
       8    composeStart,
       9    composeStop,
      10    composeUp,
      11 +  createComposeCleanupTarget,                                  
      12    getFreePort,
      13    isDockerAvailableSync,
      14 +  listAllVolumeNames,                                          
      15 +  resolveComposePersistence,                                   
      16    uniqueProjectName,
      17    waitForHealthy,
      18    waitForHttpStatus,

● Update(tests\integration\docker-compose\proxy-health.test.ts)
  ⎿  Added 57 lines, removed 3 lines
      63    const project: ComposeProject = { projectName: uniqueProjectN
          ame("relis-proxy-health"), composeFile, cwd: repoRoot };
      64  
      65    let nginxPort: number;
      66 +  let composeEnv: Record<string, string>;                      
      67 +  /** Volumes that already existed on the daemon before this te
         +st ran — must all survive it. See sub-issue 02.03's guarded-cle
         +anup contract. */                                              
      68 +  let baselineVolumeNames: string[] = [];                      
      69  
      70    beforeAll(async () => {
      71      // A real free port, not the ephemeral "0" placeholder: bak
          ed into
     ...
       74      // doesn't assert on that value directly, but an invalid b
           aked-in URL
       75      // would still leave `web` itself servable, which is exerc
           ised below.
       76      nginxPort = await getFreePort();
       74 -    const env = { NGINX_HTTP_PORT: String(nginxPort) };       
       77 +    composeEnv = { NGINX_HTTP_PORT: String(nginxPort) };      
       78  
       76 -    const up = await composeUp(project, env, ["nginx"], { buil
          -d: true, timeoutMs: 300000 });                                
       79 +    // Baseline FIRST, before anything is provisioned — see   
       80 +    // persistence.test.ts (sub-issue 02.03) for why: the post
          +-cleanup                                                      
       81 +    // "nothing unrelated was destroyed" check needs a genuine
          +ly                                                            
       82 +    // untouched snapshot, and the safety gate immediately bel
          +ow needs                                                      
       83 +    // one to compare against. This suite starts only `nginx` 
       84 +    // (web/api via depends_on) — neither mounts the project's
          + two named                                                    
       85 +    // volumes (`postgres-data`/`storage-data`), so no volume 
          +is actually                                                   
       86 +    // expected to be created — but the SAME guarded gate/clea
          +nup                                                           
       87 +    // contract applies regardless, rather than bypassing it b
          +ecause                                                        
       88 +    // "this one shouldn't create anything."                  
       89 +    baselineVolumeNames = await listAllVolumeNames();         
       90 +                                                              
       91 +    const resolved = await resolveComposePersistence(project, 
          +composeEnv);                                                  
       92 +    // Hard safety gate — BEFORE a single container is created
          +: the                                                         
       93 +    // resolved configuration must be genuinely disposable (vo
          +lume names                                                    
       94 +    // scoped to this project, no `external: true`, nothing al
          +ready on                                                      
       95 +    // the daemon under these exact names). Throws and stops s
          +hort of                                                       
       96 +    // provisioning anything if it fails.                     
       97 +    assertProvisioningSafety(resolved, baselineVolumeNames);  
       98 +                                                              
       99 +    const up = await composeUp(project, composeEnv, ["nginx"],
          + { build: true, timeoutMs: 300000 });                         
      100      if (up.exitCode !== 0) {
      101        throw new Error(`docker compose up failed (exit ${up.exi
           tCode}):\n${up.stdout}\n${up.stderr}`);
      102      }
     ...
      104    }, 300000);
      105  
      106    afterAll(async () => {
       84 -    await cleanupComposeProject(project, { timeoutMs: 180000 }
          -);                                                            
      107 +    // Guarded cleanup (sub-issue 02.03's contract), not a bar
          +e                                                             
      108 +    // `cleanupComposeProject`: `createComposeCleanupTarget` r
          +e-resolves                                                    
      109 +    // the exact configured volume names FRESH (never a cached
          + value) and                                                   
      110 +    // unions them with whatever the daemon's own `com.docker.
          +compose.project`                                              
      111 +    // label filter additionally reports, then `cleanupDisposa
          +bleTargets`                                                   
      112 +    // re-verifies ownership for every one of them immediately
          + before any                                                   
      113 +    // deletion — never relying on the safety gate run in `bef
          +oreAll`.                                                      
      114 +    // `composeEnv` is passed through unchanged so configurati
          +on                                                            
      115 +    // resolution, the original `up`, and this cleanup all res
          +olve the                                                      
      116 +    // SAME configuration. A failure discovering volumes (fail
          +s closed)                                                     
      117 +    // or an ownership refusal blocks the destructive step for
          + this                                                         
      118 +    // target and falls back to removing only containers/netwo
          +rk — always                                                   
      119 +    // safe, since those belong solely to this project.       
      120 +    const target = await createComposeCleanupTarget(project, c
          +omposeEnv, baselineVolumeNames);                              
      121 +    const [outcome] = await cleanupDisposableTargets([target])
          +;                                                             
      122 +                                                              
      123 +    // Report a cleanup failure as its OWN failure — Vitest al
          +ready                                                         
      124 +    // reports a hook failure separately from any earlier test
          +/beforeAll                                                    
      125 +    // failure, so this never hides or replaces the original c
          +ause.                                                         
      126 +    const problems: string[] = [];                            
      127 +    if (outcome.error) {                                      
      128 +      problems.push(`cleanup of "${outcome.label}" failed: ${o
          +utcome.error}`);                                              
      129 +    }                                                         
      130 +    if (outcome.volumesRequiringManualInspection.length > 0) {
      131 +      problems.push(                                          
      132 +        `volume(s) were NOT deleted — ownership could not be e
          +stablished, and they require manual inspection: ` +           
      133 +          outcome.volumesRequiringManualInspection.map((v) => 
          +`${v.name} (${v.reason})`).join("; "),                        
      134 +      );                                                      
      135 +    }                                                         
      136 +    if (problems.length > 0) {                                
      137 +      throw new Error(`Cleanup did not complete correctly:\n- 
          +${problems.join("\n- ")}`);                                   
      138 +    }                                                         
      139    }, 180000);
      140  
      141    it("proxies GET /api/health and GET /api/ready to apps/api's
            existing routes, preserving status and body", async () => {

  Ran 1 shell command (ctrl+o to expand)

● Good, only a comment reference remains, no import. Now let's typecheck.

  Ran 1 shell command (ctrl+o to expand)

● Now let's apply the same fix to port-exposure.test.ts's runtime describe
  block.

  Read 1 file (ctrl+o to expand)

● This is the full-stack runtime describe that actually creates real
  postgres-data/storage-data volumes. Let's apply the guarded pattern here.

● Update(tests\integration\docker-compose\port-exposure.test.ts)
  ⎿  Added 5 lines, removed 1 line
       5    type ComposeProject,
       6    type ContainerPortBinding,
       7    assertLoopbackOnlyBindings,
       8 -  cleanupComposeProject,                                       
       8 +  assertProvisioningSafety,                                    
       9 +  cleanupDisposableTargets,                                    
      10    composePort,
      11    composeUp,
      12 +  createComposeCleanupTarget,                                  
      13    getContainerHostPortBindings,
      14    getFreePort,
      15    isDockerAvailableSync,
      16    isLoopbackOnlyHostIp,
      17 +  listAllVolumeNames,                                          
      18 +  resolveComposePersistence,                                   
      19    resolveComposePublishedPorts,
      20    uniqueProjectName,
      21    waitForExitCode,

● Update(tests\integration\docker-compose\port-exposure.test.ts)
  ⎿  Added 49 lines, removed 3 lines
      171  // -----------------------------------------------------------
           ----------
      172  describe.runIf(dockerAvailable)("real running containers — act
           ual host port bindings", () => {
      173    const project: ComposeProject = { projectName: uniqueProject
           Name("relis-port-runtime"), composeFile, cwd: repoRoot };
      174 +  let composeEnv: Record<string, string>;                     
      175 +  /** Volumes that already existed on the daemon before this t
          +est ran — must all survive it. See sub-issue 02.03's guarded-c
          +leanup contract. */                                           
      176 +  let baselineVolumeNames: string[] = [];                     
      177  
      178    beforeAll(async () => {
      179      const nginxPort = await getFreePort();
      177 -    const env = { NGINX_HTTP_PORT: String(nginxPort), MAILHOG_
          -UI_PORT: "0", SEAWEEDFS_S3_PORT: "0" };                       
      180 +    composeEnv = { NGINX_HTTP_PORT: String(nginxPort), MAILHOG
          +_UI_PORT: "0", SEAWEEDFS_S3_PORT: "0" };                      
      181  
      179 -    const up = await composeUp(project, env, [], { build: true
          -, timeoutMs: 600000 });                                       
      182 +    // Baseline FIRST, before anything is provisioned — this s
          +uite brings                                                   
      183 +    // up the FULL stack, including `postgres` and `storage`, 
          +which DO                                                      
      184 +    // create this project's two real named volumes           
      185 +    // (`postgres-data`/`storage-data`). See persistence.test.
          +ts (sub-issue                                                 
      186 +    // 02.03) for the full rationale.                         
      187 +    baselineVolumeNames = await listAllVolumeNames();         
      188 +                                                              
      189 +    const resolved = await resolveComposePersistence(project, 
          +composeEnv);                                                  
      190 +    // Hard safety gate — BEFORE a single container or volume 
          +is created:                                                   
      191 +    // the resolved configuration must be genuinely disposable
          + (volume                                                      
      192 +    // names scoped to this project, no `external: true`, noth
          +ing already                                                   
      193 +    // on the daemon under these exact names). Throws and stop
          +s short of                                                    
      194 +    // provisioning anything if it fails.                     
      195 +    assertProvisioningSafety(resolved, baselineVolumeNames);  
      196 +                                                              
      197 +    const up = await composeUp(project, composeEnv, [], { buil
          +d: true, timeoutMs: 600000 });                                
      198      if (up.exitCode !== 0) {
      199        throw new Error(`docker compose up failed (exit ${up.exi
           tCode}):\n${up.stdout}\n${up.stderr}`);
      200      }
     ...
      217    }, 600000);
      218  
      219    afterAll(async () => {
      202 -    await cleanupComposeProject(project, { timeoutMs: 180000 }
          -);                                                            
      220 +    // Guarded cleanup (sub-issue 02.03's contract): re-resolv
          +es the exact                                                  
      221 +    // configured volume names FRESH, unions them with whateve
          +r the                                                         
      222 +    // daemon's own project-label filter additionally reports,
          + and                                                          
      223 +    // re-verifies ownership for every one of them immediately
          + before any                                                   
      224 +    // deletion — never relying on the safety gate run in `bef
          +oreAll`.                                                      
      225 +    // `composeEnv` is passed through unchanged so configurati
          +on                                                            
      226 +    // resolution, the original `up`, and this cleanup all res
          +olve the                                                      
      227 +    // SAME configuration. A discovery failure (fails closed) 
          +or an                                                         
      228 +    // ownership refusal blocks the destructive step and falls
          + back to                                                      
      229 +    // removing only containers/network — always safe.        
      230 +    const target = await createComposeCleanupTarget(project, c
          +omposeEnv, baselineVolumeNames);                              
      231 +    const [outcome] = await cleanupDisposableTargets([target])
          +;                                                             
      232 +                                                              
      233 +    // Report a cleanup failure as its OWN failure — never hid
          +ing or                                                        
      234 +    // replacing an earlier test/beforeAll failure, which Vite
          +st already                                                    
      235 +    // reports separately.                                    
      236 +    const problems: string[] = [];                            
      237 +    if (outcome.error) {                                      
      238 +      problems.push(`cleanup of "${outcome.label}" failed: ${o
          +utcome.error}`);                                              
      239 +    }                                                         
      240 +    if (outcome.volumesRequiringManualInspection.length > 0) {
      241 +      problems.push(                                          
      242 +        `volume(s) were NOT deleted — ownership could not be e
          +stablished, and they require manual inspection: ` +           
      243 +          outcome.volumesRequiringManualInspection.map((v) => 
          +`${v.name} (${v.reason})`).join("; "),                        
      244 +      );                                                      
      245 +    }                                                         
      246 +    if (problems.length > 0) {                                
      247 +      throw new Error(`Cleanup did not complete correctly:\n- 
          +${problems.join("\n- ")}`);                                   
      248 +    }                                                         
      249    }, 180000);
      250  
      251    it.each(NO_HOST_PORT_SERVICES)("publishes NO host port for %
           s", async (service) => {

● Now let's typecheck and verify no stale references remain.

  Ran 1 shell command (ctrl+o to expand)

● Typecheck is clean and no stale cleanupComposeProject references remain. Now
  let's add the regression coverage for unsafe-ownership/failed-discovery
  blocking destructive cleanup, extending persistence-safety.test.ts (the
  designated fakes-based location per existing convention).

  Read 1 file (ctrl+o to expand)

● Update(tests\integration\docker-compose\persistence-safety.test.ts)
  ⎿  Added 166 lines
      585    });
      586  });
      587  
      588 +// Regression coverage for a review finding on sub-issue 02.04
          +:                                                             
      589 +// proxy-health.test.ts and port-exposure.test.ts now build th
          +eir cleanup                                                   
      590 +// via `createComposeCleanupTarget` + `cleanupDisposableTarget
          +s` (the same                                                  
      591 +// guarded contract `persistence.test.ts` uses — see this file
          +'s own                                                        
      592 +// header comment for why the NEGATIVE ownership/discovery pat
          +hs are                                                        
      593 +// proven here, with fakes, rather than against a real volume)
          +. These                                                       
      594 +// cases mirror those two files' own project shapes specifical
          +ly —                                                          
      595 +// proxy-health.test.ts starts only `nginx`/`web`/`api` (no vo
          +lume is                                                       
      596 +// actually expected to exist), port-exposure.test.ts's runtim
          +e suite                                                       
      597 +// starts the full stack (both `postgres-data`/`storage-data` 
          +really                                                        
      598 +// exist) — proving the SAME guard rejects destructive cleanup
          + for both                                                     
      599 +// shapes when ownership is unsafe or discovery itself fails. 
      600 +describe("guarded cleanup for the 02.04 proxy-health/port-expo
          +sure call sites — unsafe ownership or failed discovery blocks 
          +destruction", () => {                                         
      601 +  function fakeGuardedTarget(options: {                       
      602 +    label: string;                                            
      603 +    resolveConfiguredVolumeNames: () => Promise<string[]>;    
      604 +    listLabeledVolumeNames: () => Promise<string[]>;          
      605 +    inspectVolumeByName: (name: string) => Promise<VolumeOwner
          +ship | undefined>;                                            
      606 +    context: VolumeOwnershipContext;                          
      607 +  }): DisposableCleanupTarget & { keepCalls: number; destroyCa
          +lls: number } {                                               
      608 +    const calls = { keep: 0, destroy: 0 };                    
      609 +    return {                                                  
      610 +      label: options.label,                                   
      611 +      discoverVolumes: () =>                                  
      612 +        discoverVolumesRequiringInspection({                  
      613 +          resolveConfiguredVolumeNames: options.resolveConfigu
          +redVolumeNames,                                               
      614 +          listLabeledVolumeNames: options.listLabeledVolumeNam
          +es,                                                           
      615 +          inspectVolumeByName: options.inspectVolumeByName,   
      616 +        }),                                                   
      617 +      ownershipContext: options.context,                      
      618 +      removeContainersKeepingVolumes: async () => {           
      619 +        calls.keep += 1;                                      
      620 +      },                                                      
      621 +      removeContainersAndVolumes: async () => {               
      622 +        calls.destroy += 1;                                   
      623 +      },                                                      
      624 +      get keepCalls() {                                       
      625 +        return calls.keep;                                    
      626 +      },                                                      
      627 +      get destroyCalls() {                                    
      628 +        return calls.destroy;                                 
      629 +      },                                                      
      630 +    };                                                        
      631 +  }                                                           
      632 +                                                              
      633 +  it("proxy-health.test.ts's shape — a FAILED configuration re
          +solution (e.g. 'docker compose config' itself failing) fails c
          +losed and never destroys anything", async () => {             
      634 +    const target = fakeGuardedTarget({                        
      635 +      label: "relis-proxy-health-aaaa1111",                   
      636 +      resolveConfiguredVolumeNames: async () => {             
      637 +        throw new Error("docker compose config --format json f
          +ailed (exit 1)");                                             
      638 +      },                                                      
      639 +      listLabeledVolumeNames: async () => [],                 
      640 +      inspectVolumeByName: async () => fakeVolumeOwnership(), 
      641 +      context: { expectedProjectName: "relis-proxy-health-aaaa
          +1111", baselineVolumeNames: [], externalVolumeKeys: [] },     
      642 +    });                                                       
      643 +                                                              
      644 +    const [outcome] = await cleanupDisposableTargets([target])
          +;                                                             
      645 +    expect(outcome.error).toMatch(/docker compose config.*fail
          +ed/i);                                                        
      646 +    expect(outcome.volumesDeleted).toEqual([]);               
      647 +    expect(target.destroyCalls).toBe(0);                      
      648 +    // Setup never completed, so even the safe containers-only
          + removal is                                                   
      649 +    // never attempted for this target either — there is nothi
          +ng it owns                                                    
      650 +    // to clean up.                                           
      651 +    expect(target.keepCalls).toBe(0);                         
      652 +  });                                                         
      653 +                                                              
      654 +  it("proxy-health.test.ts's shape — neither postgres-data nor
          + storage-data was ever created (both CONFIRMED absent): cleanu
          +p proceeds safely with nothing to delete", async () => {      
      655 +    const projectName = "relis-proxy-health-bbbb2222";        
      656 +    const target = fakeGuardedTarget({                        
      657 +      label: projectName,                                     
      658 +      resolveConfiguredVolumeNames: async () => [`${projectNam
          +e}_postgres-data`, `${projectName}_storage-data`],            
      659 +      listLabeledVolumeNames: async () => [],                 
      660 +      inspectVolumeByName: async () => undefined,             
      661 +      context: { expectedProjectName: projectName, baselineVol
          +umeNames: [], externalVolumeKeys: [] },                       
      662 +    });                                                       
      663 +                                                              
      664 +    const [outcome] = await cleanupDisposableTargets([target])
          +;                                                             
      665 +    expect(outcome.volumesDeleted).toEqual([]);               
      666 +    expect(outcome.volumesRequiringManualInspection).toEqual([
          +]);                                                           
      667 +    // A CONFIRMED absence is never a reason to block the dest
          +ructive                                                       
      668 +    // step — there is simply nothing there to delete or prote
          +ct.                                                           
      669 +    expect(target.destroyCalls).toBe(1);                      
      670 +    expect(target.keepCalls).toBe(0);                         
      671 +  });                                                         
      672 +                                                              
      673 +  it("port-exposure.test.ts's shape — a MISLABELED postgres-da
          +ta (real daemon volume, wrong/missing Compose project label) R
          +EFUSES the destructive step for the whole target", async () =>
          + {                                                            
      674 +    const projectName = "relis-port-runtime-cccc3333";        
      675 +    const postgresDataName = `${projectName}_postgres-data`;  
      676 +    const storageDataName = `${projectName}_storage-data`;    
      677 +    const target = fakeGuardedTarget({                        
      678 +      label: projectName,                                     
      679 +      resolveConfiguredVolumeNames: async () => [postgresDataN
          +ame, storageDataName],                                        
      680 +      listLabeledVolumeNames: async () => [storageDataName],  
      681 +      inspectVolumeByName: async (name) => {                  
      682 +        if (name === postgresDataName) {                      
      683 +          // Exists physically (Compose created it, by name) b
          +ut its                                                        
      684 +          // ownership label does not match this disposable pr
          +oject —                                                       
      685 +          // exactly the "mislabeled or relabeled volume" scen
          +ario                                                          
      686 +          // `discoverVolumesRequiringInspection`'s union is m
          +eant to                                                       
      687 +          // still catch, even though label-based discovery al
          +one missed it.                                                
      688 +          return fakeVolumeOwnership({ name: postgresDataName,
          + composeProject: "someone-elses-dev-stack", composeVolumeKey: 
          +"postgres-data" });                                           
      689 +        }                                                     
      690 +        return fakeVolumeOwnership({ name: storageDataName, co
          +mposeProject: projectName, composeVolumeKey: "storage-data" })
          +;                                                             
      691 +      },                                                      
      692 +      context: { expectedProjectName: projectName, baselineVol
          +umeNames: [], externalVolumeKeys: [] },                       
      693 +    });                                                       
      694 +                                                              
      695 +    const [outcome] = await cleanupDisposableTargets([target])
          +;                                                             
      696 +    expect(outcome.volumesDeleted).toEqual([]);               
      697 +    expect(outcome.volumesRequiringManualInspection).toEqual([
      698 +      { name: postgresDataName, reason: expect.stringContainin
          +g("someone-elses-dev-stack") },                               
      699 +    ]);                                                       
      700 +    // The destructive command is NEVER called for this target
          + — only the                                                   
      701 +    // safe, containers-only removal runs, for BOTH volumes to
          +gether                                                        
      702 +    // (ownership is all-or-nothing per target).              
      703 +    expect(target.destroyCalls).toBe(0);                      
      704 +    expect(target.keepCalls).toBe(1);                         
      705 +  });                                                         
      706 +                                                              
      707 +  it("port-exposure.test.ts's shape — storage-data already exi
          +sted on the daemon before this run (the baseline) REFUSES dest
          +ruction even though postgres-data is genuinely owned", async (
          +) => {                                                        
      708 +    const projectName = "relis-port-runtime-dddd4444";        
      709 +    const postgresDataName = `${projectName}_postgres-data`;  
      710 +    const storageDataName = `${projectName}_storage-data`;    
      711 +    const target = fakeGuardedTarget({                        
      712 +      label: projectName,                                     
      713 +      resolveConfiguredVolumeNames: async () => [postgresDataN
          +ame, storageDataName],                                        
      714 +      listLabeledVolumeNames: async () => [postgresDataName, s
          +torageDataName],                                              
      715 +      inspectVolumeByName: async (name) =>                    
      716 +        fakeVolumeOwnership({ name, composeProject: projectNam
          +e, composeVolumeKey: name === postgresDataName ? "postgres-dat
          +a" : "storage-data" }),                                       
      717 +      context: { expectedProjectName: projectName, baselineVol
          +umeNames: [storageDataName], externalVolumeKeys: [] },        
      718 +    });                                                       
      719 +                                                              
      720 +    const [outcome] = await cleanupDisposableTargets([target])
          +;                                                             
      721 +    expect(outcome.volumesDeleted).toEqual([]);               
      722 +    expect(outcome.volumesRequiringManualInspection).toEqual([
          +{ name: storageDataName, reason: expect.stringMatching(/pre-ex
          +isting/i) }]);                                                
      723 +    expect(target.destroyCalls).toBe(0);                      
      724 +    expect(target.keepCalls).toBe(1);                         
      725 +  });                                                         
      726 +                                                              
      727 +  it("port-exposure.test.ts's shape — an INSPECTION FAILURE fo
          +r one volume (daemon unreachable mid-discovery) REFUSES destru
          +ction for the whole target, never reading it as absence", asyn
          +c () => {                                                     
      728 +    const projectName = "relis-port-runtime-eeee5555";        
      729 +    const postgresDataName = `${projectName}_postgres-data`;  
      730 +    const storageDataName = `${projectName}_storage-data`;    
      731 +    const target = fakeGuardedTarget({                        
      732 +      label: projectName,                                     
      733 +      resolveConfiguredVolumeNames: async () => [postgresDataN
          +ame, storageDataName],                                        
      734 +      listLabeledVolumeNames: async () => [postgresDataName, s
          +torageDataName],                                              
      735 +      inspectVolumeByName: async (name) => {                  
      736 +        if (name === storageDataName) {                       
      737 +          throw new Error("daemon unreachable while inspecting
          + storage-data");                                              
      738 +        }                                                     
      739 +        return fakeVolumeOwnership({ name: postgresDataName, c
          +omposeProject: projectName, composeVolumeKey: "postgres-data" 
          +});                                                           
      740 +      },                                                      
      741 +      context: { expectedProjectName: projectName, baselineVol
          +umeNames: [], externalVolumeKeys: [] },                       
      742 +    });                                                       
      743 +                                                              
      744 +    const [outcome] = await cleanupDisposableTargets([target])
          +;                                                             
      745 +    expect(outcome.volumesDeleted).toEqual([]);               
      746 +    expect(outcome.volumesRequiringManualInspection).toEqual([
      747 +      { name: storageDataName, reason: expect.stringContaining
          +("daemon unreachable while inspecting storage-data") },       
      748 +    ]);                                                       
      749 +    expect(target.destroyCalls).toBe(0);                      
      750 +    expect(target.keepCalls).toBe(1);                         
      751 +  });                                                         
      752 +});                                                           
      753 +                                                              
      754  describe("parseS3ErrorBody / isS3ObjectAbsent — a specific S3 
           'not found', never merely 'any error'", () => {
      755    const NO_SUCH_KEY_BODY =
      756      '<?xml version="1.0" encoding="UTF-8"?>\n<Error><Code>NoSu
           chKey</Code><Message>The specified key does not exist.</Messag
           e></Error>';

● Now let's typecheck and run this file.

  Ran 2 shell commands (ctrl+o to expand)

● All 48 pass (5 new regression tests added). Now let's run the two real Docker
  suites end to end with the corrected safeguards.

  Ran 2 shell commands (ctrl+o to expand)

● Both pass with the guarded cleanup. Let's confirm clean teardown (no leaked
  containers/volumes/networks) and then run the full integration suite +
  wait-for-http-status + persistence-safety together for a final regression
  pass.

  Ran 1 shell command (ctrl+o to expand)

● Clean. Now the full integration suite for a final regression check.

  Ran 1 shell command (ctrl+o to expand)

● All 25 files pass (324 passed, 8 skipped expected "blocked" branches, 0
  failed). Let's do a final leak check and full typecheck/lint sweep (excluding
  the pre-existing unrelated web lint issue), then check git status.

  Ran 1 shell command (ctrl+o to expand)

● Clean. Let's check the final diff/status to confirm scope stayed limited to
  the correction.

  Ran 3 shell commands (ctrl+o to expand)

● The apps/web/tsconfig.json auto-patch reappeared as an incidental side effect
  of tooling in this environment (same Next.js-style reformatting as before).
  It's unrelated to this fix, so I'll revert it again.

  Ran 1 shell command (ctrl+o to expand)

● Now let's update the documentation to reflect the guarded-cleanup correction
  and the new wait-for-http-status.test.ts coverage.

  Read 1 file, ran 1 shell command (ctrl+o to expand)
● Now let's update these two bullets to mention guarded cleanup, and add bullets
  for the new wait-for-http-status.test.ts and the persistence-safety.test.ts
  extension.

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 25 lines, removed 2 lines
      622    false `200` or the web page, while `/` (served by the untouc
           hed `web`
      623    container) and `/nginx-health` keep working; and restarting 
           `api`
      624    recovers within the existing success contract, confirmed by 
           bounded
      625 -  polling (`waitForHttpStatus`).                              
      625 +  polling (`waitForHttpStatus`). Setup/teardown follow the SAM
          +E guarded                                                     
      626 +  lifecycle contract as `persistence.test.ts` (sub-issue 02.03
          +) — see                                                       
      627 +  "Cleanup robustness" below — even though this suite's own se
          +rvices                                                        
      628 +  (`nginx`/`web`/`api`) are not expected to create either name
          +d volume.                                                     
      629  - `port-exposure.test.ts` — implements local sub-issue
      630    [02.04](../../../context/notion-tasks/sub-issues/02-04.md)'s
            AC2 at three
      631    layers: (1) pure, Docker-free unit tests of the loopback-onl
           y validation
     ...
      642    migration services, real-but-exited) containers' own `docker
            inspect`
      643    `HostConfig.PortBindings` (`getContainerHostPortBindings`) —
            the actual
      644    runtime evidence the resolved configuration alone cannot pro
           vide,
      642 -  confirming every service matches layer (2) in practice.     
      645 +  confirming every service matches layer (2) in practice. Laye
          +r (3) brings                                                  
      646 +  up the full stack, including `postgres`/`storage` (which DO 
          +create this                                                   
      647 +  project's two real named volumes), so its setup/teardown use
          + the SAME                                                     
      648 +  guarded lifecycle contract as `persistence.test.ts` — baseli
          +ne capture,                                                   
      649 +  `assertProvisioningSafety` before `up`, and `createComposeCl
          +eanupTarget`                                                  
      650 +  + `cleanupDisposableTargets` in `afterAll` (see "Cleanup rob
          +ustness"                                                      
      651 +  below) — rather than an unguarded `cleanupComposeProject`.  
      652 +- `wait-for-http-status.test.ts` — deterministic, Docker-free 
          +regression                                                    
      653 +  coverage for `waitForHttpStatus` itself (used by `proxy-heal
          +th.test.ts`'s                                                 
      654 +  recovery check above), against a real local `node:http` serv
          +er                                                            
      655 +  (loopback, ephemeral port — never an external service): reso
          +lves on an                                                    
      656 +  acceptable status, retries past repeated unacceptable ones a
          +nd recovers,                                                  
      657 +  times out reporting the last observed status, and — the regr
          +ession this                                                   
      658 +  file exists for — stays genuinely BOUNDED by `timeoutMs` eve
          +n when a                                                      
      659 +  single request hangs and never responds at all (an earlier v
          +ersion only                                                   
      660 +  checked the deadline *between* requests, so one hanging requ
          +est could                                                     
      661 +  keep the whole wait open indefinitely — exactly the shape of
          + nginx's own                                                  
      662 +  60s `proxy_connect_timeout` behavior documented above). A de
          +dicated case                                                  
      663 +  confirms the pending request is actually cancelled at the ne
          +twork level                                                   
      664 +  (observed via the test server's own `req.on("close")`), not 
          +merely that                                                   
      665 +  the calling promise gives up locally.                       
      666  - `dependency-unavailable.test.ts` — explicitly starts `postgr
           es` (plus
      667    `nginx`/`api`; an earlier version only requested `["nginx"]`
           , which
      668    never actually started `postgres` at all, since nothing in n
           ginx's own

● Now let's add a mention of the new regression cases to the
  persistence-safety.test.ts/Cleanup-robustness documentation, and update the
  "Cleanup robustness" section slightly to note the two new call sites.

  Read 1 file (ctrl+o to expand)

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 20 lines
      772  cleanup call is still scoped to exactly its own uniquely-named
            project
      773  (`docker compose -p <project> down --volumes --remove-orphans`
           ), never
      774  touching another project's containers, network, or volumes.
      775 +                                                              
      776 +`proxy-health.test.ts` and `port-exposure.test.ts` (sub-issue 
          +02.04) use                                                    
      777 +the FULLER guarded contract instead — the same one `persistenc
          +e.test.ts`                                                    
      778 +uses (below): a volume baseline captured before anything is pr
          +ovisioned,                                                    
      779 +`assertProvisioningSafety` checked before `composeUp`, and    
      780 +`createComposeCleanupTarget` + `cleanupDisposableTargets` in `
          +afterAll`                                                     
      781 +(re-resolving the exact configured volume names fresh and re-v
          +erifying                                                      
      782 +ownership immediately before any deletion, failing closed on a
          + discovery                                                    
      783 +failure, and falling back to a containers-only removal — never
          + the                                                          
      784 +destructive one — when ownership cannot be established). An ea
          +rlier                                                         
      785 +version of both files called `cleanupComposeProject` directly,
          + bypassing                                                    
      786 +this contract; `persistence-safety.test.ts` now also carries f
          +akes-based                                                    
      787 +regression coverage, shaped specifically after these two files
          +' own                                                         
      788 +projects (a `nginx`/`web`/`api`-only project with no volume ex
          +pected to                                                     
      789 +exist, and a full-stack project that genuinely owns `postgres-
          +data`/                                                        
      790 +`storage-data`), proving a failed discovery or an unsafe owner
          +ship check                                                    
      791 +(mislabeled, pre-existing/baseline, or an inspection failure) 
          +still blocks                                                  
      792 +the destructive step for either shape — never exercised agains
          +t a real                                                      
      793 +volume, for the same reason `persistence.test.ts` itself doesn
          +'t (see that                                                  
      794 +file's own header comment).                                   
      795  `config-validation.test.ts` previously had no cleanup step at 
           all, even
      796  though `composeRun`'s `--rm` only removes the one-off containe
           r it
      797  creates — the project's implicitly-created network (Compose cr
           eates this

● Now let's confirm the final file list and re-run the complete regression
  battery once more as a final sanity check.

  Ran 3 shell commands (ctrl+o to expand)

● No leaked Docker resources. The tsconfig.json side effect reappeared from the
  full suite run — reverting once more as the final step.

  Ran 1 shell command (ctrl+o to expand)

● Summary — both review findings fixed, scope limited to the corrections

  Finding 1 — guarded lifecycle restored

  - proxy-health.test.ts and port-exposure.test.ts's runtime suite now follow
    the exact sub-issue 02.03 contract instead of calling cleanupComposeProject
    directly:
    - Capture baselineVolumeNames via listAllVolumeNames() before provisioning.
    - assertProvisioningSafety(resolved, baselineVolumeNames) on the resolved
      configuration before composeUp.
    - composeEnv is one value threaded through resolution → composeUp →
      createComposeCleanupTarget → cleanup (never a different environment at any
      stage).
    - afterAll builds a target with createComposeCleanupTarget and tears down
      via cleanupDisposableTargets, which re-resolves volume names fresh,
      re-verifies ownership immediately before deletion, fails closed on
      discovery failure, and falls back to containers-only removal when
      ownership can't be established.
    - Cleanup problems (outcome.error, volumesRequiringManualInspection) are
      thrown as an additional failure — Vitest reports this separately, never
      hiding an earlier test/beforeAll failure.
  - Added 5 new fakes-based regression tests to persistence-safety.test.ts,
    shaped specifically after these two files' own project types (no-volume
    nginx/web/api project; full-stack postgres-data/storage-data project):
    failed discovery, mislabeled volume, pre-existing baseline volume, and
    inspection failure all block the destructive step — never exercised against
    a real volume, consistent with persistence.test.ts's own documented stance.

  Finding 2 — waitForHttpStatus genuinely bounded

  - Rewrote it to compute one overall deadline; each fetch gets an
    AbortController tied to the remaining time, and the retry delay is clamped
    the same way — a single hanging request can no longer push the wait past
    timeoutMs.
  - Rejected (unacceptable) responses have their body released
    (response.body?.cancel()); the accepted response is returned untouched for
    the caller.
  - New wait-for-http-status.test.ts (deterministic, local node:http server, no
    external services): acceptable-status resolution, retry-then-recover,
    timeout-with-last-status, a genuinely bounded hang (elapsed stays near
    timeoutMs, not unbounded), real network-level cancellation (observed via the
    server's own req.on("close")), and non-stalling of subsequent polls.

  Commands run (from relis/)

  - pnpm run typecheck:tests → pass (run after each edit).
  - npx vitest run --config vitest.integration.config.ts 
    tests/integration/docker-compose/wait-for-http-status.test.ts → 6 passed.
  - npx vitest run --config vitest.integration.config.ts 
    tests/integration/docker-compose/persistence-safety.test.ts → 48 passed (5
    new).
  - npx vitest run --config vitest.integration.config.ts 
    tests/integration/docker-compose/proxy-health.test.ts → 5 passed, 1 skipped
    (unique project relis-proxy-health-*).
  - npx vitest run --config vitest.integration.config.ts 
    tests/integration/docker-compose/port-exposure.test.ts → 25 passed, 1 
    skipped (unique projects relis-port-config-*/relis-port-runtime-*).
  - Full suite: npx vitest run --config vitest.integration.config.ts → 25 files,
    324 passed, 8 skipped (expected "blocked" branches), 0 failed.
  - pnpm run typecheck (repo-wide) → pass.
  - docker ps -a / volume ls / network ls --filter "name=relis-" after every
    real run → empty each time (no leaked containers/volumes/networks).

  Limitations / out of scope

  - Did not re-exercise the negative ownership/discovery paths against a real
    Docker volume — same safety stance persistence.test.ts documents;
    fakes-based coverage is the deliberate substitute.
  - An environment side-effect (Next.js-style auto-reformatting of
    apps/web/tsconfig.json, triggered by tooling in this session, unrelated to
    this fix) appeared twice and was reverted both times via git checkout —
    final git status shows only the intended files changed.
  - Did not commit, push, prune, restart Docker, or touch any existing user
    stack.

✻ Churned for 26m 39s · done 16:03

※ recap: Sub-issue 02.04 (proxy health/readiness, port exposure) is implemented and both review findings are fixed and tested. Next: awaiting your review before committing. (disable recaps in /config)

❯ Complete sub-issue 02.05: Document startup, diagnostics, and local access to  
  mail capture and job observability.                                           
                                                                                
  Work only inside relis/. Follow the repository instructions, applicable local 
  Claude skills, and the mandatory context/project-structure.md,                
  context/stack.yml, and context/testing-policy.md. Do not browse external      
  sources; the task requirements are provided below.                            
                                                                                
  Objective                                                                     
  Provide a reproducible local-development guide that another developer can     
  follow to start the existing Compose stack, inspect its state, access         
  development tools, diagnose common failures, and shut it down safely.         
                                                                                
  Existing implementation                                                       
  - The stack already includes web, API, worker, PostgreSQL, migration          
  services, Nginx, MailHog, and SeaweedFS.                                      
  - Startup, persistence/reset, proxy health/readiness, and port exposure       
  already have implementation and verification.                                 
  - README.md and docs/architecture/docker-compose-stack.md already contain     
  substantial documentation. Reuse and reconcile it rather than creating a      
  competing guide.                                                              
  - Nginx exposes the web application and /api/health and /api/ready.           
  - /nginx-health describes proxy liveness only.                                
  - MailHog UI and SeaweedFS S3 are loopback-bound development endpoints.       
  - The worker currently has no implemented queue consumer or job dashboard. A  
  running watch process is not evidence of job processing.                      
  - The queue technology remains undecided. Do not select or implement one.     
                                                                                
  Verify these observations against the current checkout before editing.        
                                                                                
  Required documentation                                                        
                                                                                
  1. Reproducible startup                                                       
  - State the working directory and prerequisites, deriving required versions   
  from repository configuration.                                                
  - Explain which host tools are required for Docker-only startup versus        
  running tests or applications directly.                                       
  - Explain environment-file setup, safe development defaults, and configurable 
  host ports. Distinguish Compose interpolation from application environment    
  loading.                                                                      
  - Document the single startup command and expected outcomes: long-running     
  services, successful one-shot migrations, and services without healthchecks.  
  - Do not require host-installed dependencies or prebuilt artifacts unless the 
  actual startup path requires them.                                            
  - Explain initial image downloads/build time without promising a fixed        
  duration.                                                                     
                                                                                
  2. Access and diagnostics                                                     
  - Provide a concise table of actual host-facing URLs, internal-only           
  addresses, purposes, and limitations.                                         
  - Verify MailHog's UI is really the mail-capture interface, not merely an     
  arbitrary HTTP 200.                                                           
  - Distinguish MailHog availability from application email delivery: do not    
  claim an application mail adapter exists without evidence.                    
  - Document scoped commands for service status, logs, migration exit codes,    
  health/readiness probes, and port inspection.                                 
  - Explain the existing readiness contract honestly: network/CORS              
  initialization is not database or queue readiness.                            
  - Cover actionable diagnosis for Docker being unavailable, occupied ports,    
  failed builds, rejected configuration, failed migrations, and unavailable API 
  upstream.                                                                     
  - Do not automatically restart Docker, delete resources, or recommend broad   
  pruning as routine troubleshooting.                                           
                                                                                
  3. Job observability                                                          
  - Inspect whether any job-observability endpoint actually exists.             
  - If none exists, state that clearly and provide only the current worker      
  status/log commands.                                                          
  - Explicitly state that worker/watch-process liveness does not prove jobs are 
  consumed or completed.                                                        
  - Report missing queue processing/dashboard integration as unavailable; do    
  not invent a URL or implement it to satisfy documentation.                    
  - Mark mail and job tooling as development-only, not product features.        
                                                                                
  4. Safe shutdown and reset                                                    
  - Reuse the existing lifecycle documentation.                                 
  - Clearly distinguish stop/start, container removal preserving data, and      
  destructive volume reset.                                                     
  - Keep destructive instructions explicitly project-scoped, with warnings      
  identifying affected data.                                                    
  - Preserve all existing ownership, baseline-volume, fail-closed, and          
  environment-consistency safeguards.                                           
                                                                                
  Verification                                                                  
  - Follow the documented startup and diagnostic workflow using a uniquely      
  named disposable project and non-conflicting ports.                           
  - Verify clean-start assumptions without relying silently on existing         
  node_modules, generated config output, or persistent volumes. If a truly      
  clean checkout was not used, state exactly what was and was not validated.    
  - Do not reset or clean the user's checkout to simulate a clean environment.  
  - Verify the application/proxy responses, MailHog identity, migration         
  outcomes, and documented endpoint exposure.                                   
  - Reuse existing automated coverage. Add tests only for meaningful uncovered  
  behavior, in the prescribed locations.                                        
  - Use guarded provisioning and cleanup for all disposable resources, with the 
  same environment throughout. Report cleanup failures.                         
  - Do not touch existing user stacks, prune resources, or restart Docker       
  Desktop.                                                                      
  - If required infrastructure is unavailable, report a blocker rather than     
  claiming validation passed.                                                   
                                                                                
  Scope                                                                         
  This is primarily documentation and verification work. Do not implement email 
  delivery, queue consumers, job dashboards, business features, production      
  deployment, or unrelated refactors. If verification exposes a defect          
  requiring changes outside this scope, report it and request direction.        
                                                                                
  Completion report                                                             
  - Map the startup/diagnostics and mail/job-tooling requirements to            
  documentation and actual evidence.                                            
  - List changed files, exact commands, exit codes, test counts, skips, and     
  remaining limitations.                                                        
  - Distinguish real execution from mocks, inspection, and inherited evidence.  
  - Confirm no unrelated changes.                                               
  - Do not automatically close the parent issue, start another sub-issue,       
  export a session, commit, or push.                                            

  Ran 1 shell command (ctrl+o to expand)

● Good, 02.04 is merged. Let's read the sub-issue 02.05 definition and related
  docs.

  Read 6 files, ran 10 shell commands (ctrl+o to expand)

● Let's quickly probe MailHog's real HTTP responses to ground the identity check
  in actual observed behavior (not assumption), using a disposable container.

  Ran 4 shell commands (ctrl+o to expand)

● Now let's look at the exact MailHog test in stack-smoke.test.ts to strengthen
  it with real identity verification.

  Read 1 file, ran 1 shell command (ctrl+o to expand)

● Now let's strengthen this test with real identity verification against
  MailHog's distinctive HTML and API contract.

● Update(tests\integration\docker-compose\stack-smoke.test.ts)
  ⎿  Added 27 lines, removed 1 line
      275      expect(logs.stdout).toContain("ReLiS worker started");
      276    });
      277  
      278 -  it("captures mail-capture's web UI as a reachable developmen
          -t-only endpoint", async () => {                               
      278 +  it("serves MailHog's OWN mail-capture interface — not merely
          + an arbitrary HTTP 200", async () => {                        
      279 +    // Local sub-issue 02.05: a bare `200` from this port is n
          +ot evidence                                                   
      280 +    // it is actually MailHog — any web server would satisfy t
          +hat. Two                                                      
      281 +    // independent, MailHog-SPECIFIC signals are checked inste
          +ad, each                                                      
      282 +    // confirmed by direct inspection of a real `mailhog/mailh
          +og:v1.0.1`                                                    
      283 +    // container during this task:                            
      284      const mailhogPort = await composePort(project, "mailhog", 
           8025);
      285 +                                                              
      286 +    // 1. The root page is MailHog's own Angular UI shell — it
          +s <title>                                                     
      287 +    // and navbar brand both read "MailHog" (apps/web's homepa
          +ge, the only                                                  
      288 +    // other UI this stack serves, has neither).              
      289      const ui = await fetch(`http://127.0.0.1:${mailhogPort}/`)
           ;
      290      expect(ui.status).toBe(200);
      291 +    const uiBody = await ui.text();                           
      292 +    expect(uiBody).toContain("<title>MailHog</title>");       
      293 +    expect(uiBody).toContain('ng-app="mailhogApp"');          
      294 +                                                              
      295 +    // 2. MailHog's own message-listing API returns its distin
          +ctive JSON                                                    
      296 +    // shape ({total, count, start, items}) — a contract speci
          +fic to                                                        
      297 +    // MailHog's real API, not a generic page. Also proves it 
          +is                                                            
      298 +    // functioning as a mail CAPTURE interface (a queryable in
          +box), not                                                     
      299 +    // just a static page that happens to look like one. An em
          +pty `items`                                                   
      300 +    // array here is expected and correct: nothing has been se
          +nt to it in                                                   
      301 +    // this test (apps/api/apps/worker have no mail adapter — 
          +see "Known                                                    
      302 +    // limitations" — so this is deliberately NOT exercised th
          +rough the                                                     
      303 +    // application), not a sign of malfunction.               
      304 +    const api = await fetch(`http://127.0.0.1:${mailhogPort}/a
          +pi/v2/messages`);                                             
      305 +    expect(api.status).toBe(200);                             
      306 +    const inbox = (await api.json()) as { total: number; count
          +: number; start: number; items: unknown[] };                  
      307 +    expect(inbox).toEqual({ total: 0, count: 0, start: 0, item
          +s: [] });                                                     
      308    });
      309  
      310    it("performs a real, credentialed object-storage upload and 
           download round-trip", async () => {

  Read 1 file (ctrl+o to expand)

● Update(tests\integration\docker-compose\stack-smoke.test.ts)
  ⎿  Added 7 lines
      49  //     running (watcher liveness) and completed its own startup
          /config
      50  //     path (its log line) — explicitly NOT claimed as job proc
          essing;
      51  //     no queue consumer exists or is added here.
      52 +//   - MailHog's root page and message-listing API ARE MailHog'
         +s own                                                          
      53 +//     mail-capture interface (local sub-issue 02.05) — checked
         + via its                                                       
      54 +//     distinctive `<title>MailHog</title>`/`ng-app="mailhogApp
         +"` markup                                                      
      55 +//     and its exact `{total,count,start,items}` API shape, not
         + merely a                                                      
      56 +//     generic HTTP 200 (which any server would satisfy). Not e
         +xercised                                                       
      57 +//     through the application: no mail adapter exists in apps/
         +api or                                                         
      58 +//     apps/worker today, so `items` is correctly empty.       
      59  //   - A real SeaweedFS S3-gateway upload/download round-trip, 
          signed
      60  //     with the development-only credentials in
      61  //     docker/storage/s3-identities.json — NOT merely "the cont
          ainer

● Now let's typecheck and run this specific test file against real Docker.

  Ran 2 shell commands (ctrl+o to expand)

● All pass against real Docker. Let's check for leaked resources, then move on
  to the documentation work.

  Ran 1 shell command (ctrl+o to expand)

● Now let's run a short, guarded manual verification pass to capture real
  diagnostic output (ps states, port-conflict error text, config errors) to
  ground the documentation precisely.

  Ran 1 shell command (ctrl+o to expand)

● Let's wait for health/readiness and capture the real docker compose ps output
  and endpoint checks.

  Ran 4 shell commands (ctrl+o to expand)

● Clean. Now let's guard-check and tear down the main verification project.

  Ran 1 shell command (ctrl+o to expand)

● Ownership confirmed exactly for both volumes — safe to destroy. Now the scoped
  teardown.

  Ran 1 shell command (ctrl+o to expand)
● Confirmed clean — no leaked containers, volumes, or networks. Now let's also
  capture a rejected-configuration error for documentation (invalid Compose
  config) and check docker info output format for the "Docker unavailable"
  scenario documentation.

  Ran 1 shell command (ctrl+o to expand)

● Now I have solid grounded evidence. Let's write the documentation. First,
  update the document header and Prerequisites section.

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 5 lines, removed 2 lines
       4  configuration, including the Control DB and per-project test
       5  databases](../../../context/notion-tasks/sub-issues/02-02.md),
       6  [02.03 — Configure local persistent volumes and exclude them fr
          om version
       7 -control](../../../context/notion-tasks/sub-issues/02-03.md), an
         -d                                                              
       7 +control](../../../context/notion-tasks/sub-issues/02-03.md),   
       8  [02.04 — Expose health/readiness through the local proxy and re
          strict
       9 -database and storage port exposure](../../../context/notion-tas
         -ks/sub-issues/02-04.md)                                        
       9 +database and storage port exposure](../../../context/notion-tas
         +ks/sub-issues/02-04.md),                                       
      10 +and [02.05 — Document startup, diagnostics, and local access to
      11 +MailHog/Mailpit and job                                        
      12 +observability](../../../context/notion-tasks/sub-issues/02-05.m
         +d)                                                             
      13  (parent: [02 — Add Docker Compose local
      14  stack](../../../context/notion-tasks/02-docker-compose.md)).
      15  Builds directly on [the local-stack inventory](local-stack-inve
          ntory.md)

● Now let's expand the Prerequisites section and add the Compose-interpolation
  vs. application-env-loading distinction.

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 70 lines, removed 12 lines
       52  
       53  ## Prerequisites
       54  
       55 -- Docker Engine with the Compose V2 CLI plugin (the `docker co
          -mpose`                                                        
       56 -  subcommand, not the legacy standalone `docker-compose` V1 bi
          -nary), with                                                   
       57 -  a Linux-container backend. **Verified by real execution** (D
          -ocker                                                         
       58 -  Desktop 29.8.2 / `docker compose` plugin v5.5.1, Windows hos
          -t,                                                            
       59 -  WSL2/Linux-container engine) — see the final report for sub-
          -issue #47                                                     
       60 -  for the exact commands and results. Earlier drafts of this d
          -ocument                                                       
       61 -  were written before Docker was available and said so explici
          -tly; that                                                     
       62 -  caveat no longer applies.                                   
       63 -- No `pnpm`/Node install needed on the host — every app contai
          -ner installs                                                  
       64 -  its own dependencies at image-build time (`pnpm install --fr
          -ozen-lockfile`                                                
       65 -  against the committed `pnpm-lock.yaml`).                    
       55 +Implements local sub-issue [02.05](../../../context/notion-tas
          +ks/sub-issues/02-05.md)'s                                     
       56 +"reproducible startup" requirement. What you actually need dep
          +ends on                                                       
       57 +which of the two local-development paths you're using — this r
          +epository                                                     
       58 +has **two independent** ones (see `README.md` "Docker Compose 
          +(local                                                        
       59 +stack)" vs. its earlier "Deployment"/native sections): running
          + the Docker                                                   
       60 +Compose stack below, or running tests/applications directly on
          + the host                                                     
       61 +(`pnpm dev`, `pnpm test`, `pnpm run test:e2e`, `node tooling/s
          +cripts/deploy.mjs`).                                          
       62 +                                                              
       63 +| | Docker-only startup (`docker compose up`) | Running tests 
          +or applications directly on the host |                        
       64 +| --- | --- | --- |                                           
       65 +| Docker Engine + Compose V2 plugin | **Required.** | Required
          + only for `pnpm run test:integration` (and the integration sui
          +tes under `tests/integration/docker-compose/`), which spawn re
          +al disposable Compose projects. Not required for `pnpm -r test
          +` (package unit suites) or running an app's own `dev`/`start` 
          +natively. |                                                   
       66 +| Node.js | **Not required on the host at all.** Each of `web`
          +/`api`/`worker`'s images installs its own Node 22 runtime (`no
          +de:22-bookworm-slim` — see each `docker/*/Dockerfile`'s `FROM`
          + line) and its own dependencies at **image-build time** (`pnpm
          + install --frozen-lockfile` against the committed `pnpm-lock.y
          +aml`, inside the container). | **Required**, matching this rep
          +ository's own `package.json` `engines.node` field: `>=22.15`. 
          +|                                                             
       67 +| pnpm | Not required on the host — each image installs its ow
          +n pnpm via Corepack during the build. | **Required**, pinned e
          +xactly by `package.json`'s `packageManager` field: `pnpm@11.17
          +.0`. |                                                        
       68 +| Playwright's Chromium browser | Not required unless you also
          + run `tests/e2e/docker-compose/` (below). | Required only for 
          +`pnpm run test:e2e` — install explicitly with `pnpm exec playw
          +right install chromium` (this repository's tooling never insta
          +lls it on your behalf automatically). |                       
       69 +                                                              
       70 +(Versions above are read directly from this repository's own  
       71 +`package.json`/Dockerfiles, not invented — re-check those file
          +s if this                                                     
       72 +document and the code ever disagree.) Docker Engine itself, th
          +e Compose                                                     
       73 +plugin, and a Linux-container backend are **verified by real e
          +xecution**                                                    
       74 +in the environment used for this sub-issue's own verification 
          +pass                                                          
       75 +(`docker --version` → Docker Engine `29.8.2`; `docker compose 
          +version` →                                                    
       76 +Compose plugin `v5.5.1`; Windows host, WSL2/Linux-container en
          +gine) — the                                                   
       77 +exact versions your own host reports may differ; what matters 
          +is the                                                        
       78 +Compose **V2** CLI plugin (the `docker compose` subcommand), n
          +ever the                                                      
       79 +legacy standalone `docker-compose` V1 binary, with a Linux-con
          +tainer                                                        
       80 +backend (Windows containers are not supported here).          
       81 +                                                              
       82  - Copy `.env.example` to `.env` at the repository root (same c
           onvention as
       83    the existing runtime-config setup — see `README.md` "Runtime
       68 -  configuration"). Every value in it is a development-only pla
          -ceholder.                                                     
       84 +  configuration"). Every value in it is a development-only pla
          +ceholder,                                                     
       85 +  safe to commit as an example and safe to use unmodified for 
          +ordinary                                                      
       86 +  local development.                                          
       87 +                                                              
       88 +### Compose interpolation vs. application environment loading 
          +— two SEPARATE mechanisms                                     
       89 +                                                              
       90 +This root `.env` file and `@relis/config`'s own `loadEnvFiles`
          + (documented                                                  
       91 +in `README.md` "Environment loading and override precedence") 
          +are **not                                                     
       92 +the same mechanism**, and confusing them is the single most co
          +mmon source                                                   
       93 +of "I changed `.env` but nothing happened" confusion with this
          + stack:                                                       
       94 +                                                              
       95 +- **Compose interpolation** (what the root `.env` actually fee
          +ds): `docker                                                  
       96 +  compose` itself reads `.env` from the current directory **on
          + the host**,                                                  
       97 +  purely to resolve `${VAR:-default}` placeholders written dir
          +ectly in                                                      
       98 +  `docker-compose.yml` — e.g. `POSTGRES_USER`, `POSTGRES_PASSW
          +ORD`,                                                         
       99 +  `NGINX_HTTP_PORT`, `MAILHOG_UI_PORT`, `SEAWEEDFS_S3_PORT`,  
      100 +  `PROJECT_TEST_DB_NAME`. This happens entirely before any con
          +tainer                                                        
      101 +  starts, and the resolved values are then baked into each ser
          +vice's                                                        
      102 +  `environment:` block as literal values.                     
      103 +- **Application environment loading** (`@relis/config`'s `load
          +EnvFiles`,                                                    
      104 +  used by `apps/api`/`apps/web`/`apps/worker`'s own entry poin
          +ts): reads                                                    
      105 +  `.env*` files from **inside the running process's own filesy
          +stem** —                                                      
      106 +  the application root and the owning package directory. Verif
          +ied by                                                        
      107 +  direct inspection of this repository's build context: `.dock
          +erignore`                                                     
      108 +  explicitly excludes `.env`/`.env.*` (keeping only `.env.exam
          +ple`) from                                                    
      109 +  every image's build context, and no service in `docker-compo
          +se.yml`                                                       
      110 +  declares an `env_file:` directive or bind-mounts the root `.
          +env` into                                                     
      111 +  any container. **The root `.env` file therefore never reache
          +s                                                             
      112 +  `loadEnvFiles` inside any of the three app containers at all
          +** — every                                                    
      113 +  value that process needs (`NODE_ENV`, `API_PORT`, `API_HOST`
          +,                                                             
      114 +  `API_CORS_ORIGIN`, `WEB_PORT`, `NEXT_PUBLIC_API_URL`) is ins
          +tead set                                                      
      115 +  directly as a literal in that service's own `environment:` b
          +lock in                                                       
      116 +  `docker-compose.yml` (itself possibly built from a Compose-i
          +nterpolated                                                   
      117 +  value, e.g. `NEXT_PUBLIC_API_URL: http://localhost:${NGINX_H
          +TTP_PORT:-8080}/api`                                          
      118 +  — one mechanism feeding the other, not the same mechanism tw
          +ice).                                                         
      119 +- **Consequence:** adding a NEW variable to your root `.env` a
          +nd expecting                                                  
      120 +  `apps/api`/`apps/web`/`apps/worker` to see it via `loadEnvFi
          +les` **will                                                   
      121 +  not work** inside this Compose stack — it would only work fo
          +r a                                                           
      122 +  natively-run process (`pnpm --filter "./apps/api" dev`, outs
          +ide Docker).                                                  
      123 +  To change what a containerized service's OWN code sees, add 
          +or change                                                     
      124 +  it in that service's `environment:` block in `docker-compose
          +.yml`                                                         
      125 +  (optionally still sourced from a Compose-interpolated `${VAR
          +}`).                                                          
      126 +                                                              
      127  - Only for the real-browser check (`tests/e2e/docker-compose/`
           ):
      128    Playwright's Chromium browser binary must be installed
      129    (`pnpm exec playwright install chromium`). **Now installed a
           nd

● Now let's expand the Startup section with grounded, real observable outcomes.

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 71 lines, removed 6 lines
      136  ## Startup
      137  
      138  ```bash
      139 +cd relis                 # the application root — every comman
          +d below assumes this working directory                        
      140 +cp .env.example .env     # once, if not already done          
      141  docker compose up --build
      142  ```
      143  
      142 -First run builds `web`, `api`, and `worker`'s images (each ins
          -talls the                                                     
      143 -full pnpm workspace and bakes an initial `@relis/config` build
          - — see                                                        
      144 -"Clean-checkout startup" below) and provisions two disposable 
          -PostgreSQL                                                    
      145 -databases via the `migrate-control` / `migrate-project` one-sh
          -ot services.                                                  
      146 -Subsequent runs can drop `--build` unless `package.json`/`pnpm
          --lock.yaml`                                                   
      147 -changed.                                                      
      144 +First run pulls four base images (`postgres:17-alpine`, `nginx
          +:1.27-alpine`,                                                
      145 +`mailhog/mailhog:v1.0.1`, `chrislusf/seaweedfs:3.71` — all **v
          +erified to                                                    
      146 +pull successfully** during this and earlier sub-issues' real  
      147 +verification) and builds `web`, `api`, and `worker`'s own thre
          +e images                                                      
      148 +locally, each of which installs the full pnpm workspace and ba
          +kes an                                                        
      149 +initial `@relis/config` build (see "Clean-checkout startup" be
          +low). How                                                     
      150 +long this takes is **not something this document promises a fi
          +xed                                                           
      151 +duration for** — it depends on your network speed (image pulls
          +, registry                                                    
      152 +reachability) and host CPU/disk speed (three `pnpm install    
      153 +--frozen-lockfile` runs), and Docker's own layer cache means a
          + *second*                                                     
      154 +run of this exact command, even after `git clean`-ing the work
          +ing tree,                                                     
      155 +can be meaningfully faster than the first if the underlying Do
          +ckerfile                                                      
      156 +layers haven't changed. Subsequent runs can drop `--build` unl
          +ess                                                           
      157 +`package.json`/`pnpm-lock.yaml` changed.                      
      158  
      159 +### What a successful startup actually looks like             
      160 +                                                              
      161 +Verified by real execution — one real `docker compose up -d --
          +build` run                                                    
      162 +against a uniquely-named disposable project during this sub-is
          +sue's own                                                     
      163 +verification pass (see "Verification" in the completion report
          +; `ps`                                                        
      164 +output trimmed to the relevant columns):                      
      165 +                                                              
      166 +```                                                           
      167 +NAME        STATUS                             PORTS          
      168 +api         Up 18 seconds (healthy)            3001/tcp       
      169 +mailhog     Up 17 seconds                      127.0.0.1:<MAIL
          +HOG_UI_PORT>->8025/tcp                                        
      170 +nginx       Up 16 seconds (healthy)             127.0.0.1:<NGI
          +NX_HTTP_PORT>->80/tcp                                         
      171 +postgres    Up 17 seconds (healthy)             5432/tcp      
      172 +storage     Up 18 seconds (health: starting)   127.0.0.1:<SEAW
          +EEDFS_S3_PORT>->8333/tcp                                      
      173 +web         Up 17 seconds (healthy)             3000/tcp      
      174 +worker      Up 17 seconds                       (no port, no h
          +ealth column at all)                                          
      175 +```                                                           
      176 +                                                              
      177 +A few seconds later, `storage` converges to `(healthy)` too (i
          +ts                                                            
      178 +healthcheck has a `start_period` — see "Known limitations" for
          + the two                                                      
      179 +real healthcheck defects already found and fixed here). `migra
          +te-control`                                                   
      180 +and `migrate-project` are **not** in this list at all once the
          +y've                                                          
      181 +finished — `docker compose ps` (with no `-a`) only shows curre
          +ntly-running                                                  
      182 +containers, and both are one-shot services that **exit** on su
          +ccess:                                                        
      183 +                                                              
      184 +```bash                                                       
      185 +docker compose -p relis -f docker-compose.yml ps -a migrate-co
          +ntrol migrate-project                                         
      186 +# STATUS column reads "Exited (0) ..." for both once provision
          +ing succeeds                                                  
      187 +```                                                           
      188 +                                                              
      189 +Three distinct "no health information" shapes exist, by design
          +, and all                                                     
      190 +three are expected — **none of them is a failure**:           
      191 +                                                              
      192 +- `worker` has **no `healthcheck:` block at all** in `docker-c
          +ompose.yml`                                                   
      193 +  (deliberately — see "Job observability" below for why fabric
          +ating one                                                     
      194 +  would misrepresent what this service actually does) — its `p
          +s` row                                                        
      195 +  shows no health annotation.                                 
      196 +- `mailhog` **has no `healthcheck:` block either** (its offici
          +al image is                                                   
      197 +  a near-scratch Go binary with no shell/`wget` bundled — see 
          +"Known                                                        
      198 +  limitations") — same bare "Up ..." row, no health annotation
          +.                                                             
      199 +- `migrate-control`/`migrate-project` have no `healthcheck:` e
          +ither, but                                                    
      200 +  for a different reason: they are not long-running services t
          +o probe —                                                     
      201 +  success is their own process exiting `0`.                   
      202 +                                                              
      203 +Through the proxy, immediately after `nginx` itself reports `(
          +healthy)`:                                                    
      204 +                                                              
      205 +```bash                                                       
      206 +curl http://localhost:8080/nginx-health   # -> 200, body "ok" 
          + (proxy liveness only)                                        
      207 +curl http://localhost:8080/api/health     # -> 200, {"status":
          +"ok","message":"...","service":"api"}                         
      208 +curl http://localhost:8080/api/ready      # -> 200, {"status":
          +"ok","service":"api","checks":{"network":"ok","cors":"ok"}}   
      209 +```                                                           
      210 +                                                              
      211 +(Substitute your own `${NGINX_HTTP_PORT:-8080}` if you changed
          + it.) Open                                                    
      212 +`http://localhost:8080/` in a browser for the application itse
          +lf.                                                           
      213 +                                                              
      214  ## Shutdown and reset
      215  
      216  ```bash

  Read 1 file, ran 1 shell command (ctrl+o to expand)

● Now let's insert the "Local access quick reference" table right before "##
  Health and readiness".

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 19 lines
      260    `http://localhost:${NGINX_HTTP_PORT}`) is not actually exerc
           ised in
      261    normal use.
      262  
      263 +## Local access quick reference                               
      264 +                                                              
      265 +Implements local sub-issue [02.05](../../../context/notion-tas
          +ks/sub-issues/02-05.md).                                      
      266 +Every row's "Purpose" and "Limitation" is traceable to a speci
          +fic section                                                   
      267 +below (or to "Known limitations") — this table is a map, not a
      268 +replacement for reading those sections before relying on any o
          +f this for                                                    
      269 +more than a quick lookup.                                     
      270 +                                                              
      271 +| URL / address | Host-facing or internal-only? | Purpose | Li
          +mitation |                                                    
      272 +| --- | --- | --- | --- |                                     
      273 +| `http://localhost:${NGINX_HTTP_PORT:-8080}/` | Host-facing, 
          +loopback only | The application itself (served by `web`, via n
          +ginx). | None beyond normal app behavior — not this sub-issue'
          +s concern. |                                                  
      274 +| `http://localhost:${NGINX_HTTP_PORT:-8080}/nginx-health` | H
          +ost-facing, loopback only | **Proxy liveness only.** | Does **
          +not** mean `web`/`api` are reachable or ready — see "Health an
          +d readiness" below. |                                         
      275 +| `http://localhost:${NGINX_HTTP_PORT:-8080}/api/health` | Hos
          +t-facing, loopback only | `apps/api`'s liveness route, through
          + the proxy. | Not a dependency check of any kind — see "Health
          + and readiness" below. |                                      
      276 +| `http://localhost:${NGINX_HTTP_PORT:-8080}/api/ready` | Host
          +-facing, loopback only | `apps/api`'s readiness route, through
          + the proxy. | Reports **network/CORS initialization only** — n
          +ever database or queue readiness, because `apps/api` has neith
          +er integrated. See "Health and readiness" below. |            
      277 +| `http://127.0.0.1:${MAILHOG_UI_PORT:-8025}/` | Host-facing, 
          +loopback only | MailHog's real mail-capture UI (verified ident
          +ity, not an arbitrary `200` — see "Mail capture" below). | Dev
          +elopment-only tooling, not a product feature. No application m
          +ail adapter exists to send anything to it yet — see "Mail capt
          +ure" below. |                                                 
      278 +| `http://127.0.0.1:${SEAWEEDFS_S3_PORT:-8333}/` | Host-facing
          +, loopback only | SeaweedFS's S3-compatible gateway (credentia
          +led — `docker/storage/s3-identities.json`). | Development-only
          + infrastructure. No application storage adapter exists yet. | 
      279 +| `api:3001`, `web:3000`, `postgres:5432`, `mailhog:1025` (SMT
          +P), `storage:9333`/`8888` (SeaweedFS master/filer) | **Interna
          +l only** — Compose network hostnames | Container-to-container 
          +addressing. | Never reachable from the host browser/shell at a
          +ll; reaching them requires `docker compose exec` into a contai
          +ner already on the `relis` network. See "Host port exposure po
          +licy" above for why this is deliberate, and its own explicit s
          +tatement that Docker's internal network is not a security boun
          +dary against the host administrator. |                        
      280 +| *(no URL — none exists)* | n/a | Job/queue observability. | 
          +**No such endpoint exists anywhere in this stack.** See "Job o
          +bservability" below — do not go looking for one. |            
      281 +                                                              
      282  ## Health and readiness — verified URLs and their actual meani
           ng
      283  
      284  Implements local sub-issue [02.04](../../../context/notion-tas
           ks/sub-issues/02-04.md).

● Now let's find the Host port exposure policy section end to insert the "Mail
  capture" and "Job observability" sections after it.

  Read 1 file, ran 1 shell command (ctrl+o to expand)

● Now let's insert the "Mail capture" and "Job observability" sections between
  "Host port exposure policy" and "Clean-checkout startup".

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 116 lines
      352    port. The loopback-only publish rule above is the actual, an
           d only,
      353    host-facing boundary this sub-issue establishes and verifies
           .
      354  
      355 +## Mail capture                                               
      356 +                                                              
      357 +Implements local sub-issue [02.05](../../../context/notion-tas
          +ks/sub-issues/02-05.md).                                      
      358 +MailHog is prescribed by `context/project-structure.md`'s requ
          +ired                                                          
      359 +`docker/mailhog/` entry — it is this stack's mail-capture tool
          +, not an                                                      
      360 +open MailHog-vs.-Mailpit choice still to be made (the parent t
          +ask's own                                                     
      361 +language mentions "MailHog/Mailpit" generically, but the struc
          +ture                                                          
      362 +document already settled it, and `docker-compose.yml` already 
          +runs                                                          
      363 +`mailhog/mailhog:v1.0.1`).                                    
      364 +                                                              
      365 +- **URL:** `http://127.0.0.1:${MAILHOG_UI_PORT:-8025}/` (loopb
          +ack-only —                                                    
      366 +  see "Host port exposure policy" above).                     
      367 +- **Verified to actually be MailHog's own interface, not merel
          +y an                                                          
      368 +  arbitrary `200`.** `stack-smoke.test.ts` checks two signals 
          +specific to                                                   
      369 +  the real `mailhog/mailhog:v1.0.1` image (confirmed by direct
          + inspection                                                   
      370 +  of a live container during this task), not a generic status 
          +code:                                                         
      371 +  - The root page's `<title>MailHog</title>` and `ng-app="mail
          +hogApp"` —                                                    
      372 +    markup unique to MailHog's own Angular UI shell.          
      373 +  - `GET /api/v2/messages` returns MailHog's own distinctive J
          +SON shape                                                     
      374 +    (`{"total":0,"count":0,"start":0,"items":[]}` against a fr
          +esh                                                           
      375 +    instance) — MailHog's real message-listing API, not a page
          + that                                                         
      376 +    merely looks like a mail client.                          
      377 +- **Distinguish "MailHog is running and reachable" from "the a
          +pplication                                                    
      378 +  sends mail through it" — the second does NOT exist.** Search
          +ing                                                           
      379 +  `apps/api`/`apps/web`/`apps/worker` finds no mail-sending co
          +de, no SMTP                                                   
      380 +  client dependency, and no `mail/` module anywhere in this ch
          +eckout (the                                                   
      381 +  structure document reserves such a module's location, but it
          + has not                                                      
      382 +  been created) — there is no application-level mail adapter t
          +o verify,                                                     
      383 +  and none is implemented by this sub-issue. The empty `items:
          + []` above                                                    
      384 +  is therefore the **correct**, expected state: nothing has ev
          +er been                                                       
      385 +  sent to it, by this stack or by the application.            
      386 +- **Why this task does not go further and send a real test ema
          +il through                                                    
      387 +  SMTP:** MailHog's SMTP port (`mailhog:1025`) is deliberately
      388 +  **internal-only** (see "Host port exposure policy" above) — 
          +reaching it                                                   
      389 +  would require executing a raw SMTP conversation from inside 
          +another                                                       
      390 +  container on the `relis` network (e.g. via `docker compose e
          +xec`),                                                        
      391 +  which was judged disproportionate complexity/risk for margin
          +al                                                            
      392 +  additional evidence beyond the two MailHog-specific signals 
          +above, and                                                    
      393 +  is explicitly out of this sub-issue's scope ("Do not impleme
          +nt email                                                      
      394 +  delivery"). The two checks above already distinguish MailHog
          +'s real                                                       
      395 +  identity/API from an arbitrary `200` without touching SMTP a
          +t all.                                                        
      396 +- **Development-only tooling, not a product feature.** MailHog
          + exists so                                                    
      397 +  a developer can *manually* inspect what an application would
          + have sent,                                                   
      398 +  once a mail adapter exists — it ships with this stack regard
          +less of                                                       
      399 +  whether anything is using it yet (see                       
      400 +  `docs/architecture/local-stack-inventory.md` §9 for the same
      401 +  distinction, drawn independently during the preceding sub-is
          +sue).                                                         
      402 +                                                              
      403 +## Job observability                                          
      404 +                                                              
      405 +Implements local sub-issue [02.05](../../../context/notion-tas
          +ks/sub-issues/02-05.md).                                      
      406 +**No job-observability endpoint of any kind exists in this sta
          +ck** —                                                        
      407 +there is no dashboard, no HTTP status page, and no API to quer
          +y job                                                         
      408 +state. This was verified by inspection, not assumed:          
      409 +                                                              
      410 +- `apps/worker/src/main.ts` (reproduced in full below) opens *
          +*no network                                                   
      411 +  listener at all**, in either `dev` or `start` mode — it load
          +s and                                                         
      412 +  validates configuration, logs one line, and returns:        
      413 +                                                              
      414 +  ```ts                                                       
      415 +  function main(): void {                                     
      416 +    loadEnvFiles(resolveRuntimeMode(process.env.NODE_ENV));   
      417 +    loadValidatedConfig();                                    
      418 +    console.log("ReLiS worker started");                      
      419 +  }                                                           
      420 +  ```                                                         
      421 +                                                              
      422 +- `docker-compose.yml` assigns `worker` no internal port, no p
          +ublished                                                      
      423 +  host port (independently confirmed by the real running-conta
          +iner check                                                    
      424 +  in `port-exposure.test.ts`, sub-issue 02.04), and no `health
          +check:`                                                       
      425 +  block — there is no address to query even if you wanted to. 
      426 +- No other service in this stack (`api`, `web`, `nginx`, `post
          +gres`,                                                        
      427 +  `storage`, `mailhog`) exposes anything job-related either; `
          +apps/api`'s                                                   
      428 +  `/ready` explicitly reports only `network`/`cors`, never a q
          +ueue check                                                    
      429 +  (see "Health and readiness" above).                         
      430 +                                                              
      431 +**The only observability actually available is process-level, 
          +via                                                           
      432 +Docker itself — not application-level:**                      
      433 +                                                              
      434 +```bash                                                       
      435 +docker compose -p relis -f docker-compose.yml ps worker      #
          + container lifecycle status (no health column — see "What a su
          +ccessful startup actually looks like")                        
      436 +docker compose -p relis -f docker-compose.yml logs worker    #
          + expect exactly one line: "ReLiS worker started"              
      437 +docker inspect <worker-container-id> --format '{{.State.Status
          +}}'   # "running" in dev mode (tsx watch's supervisor)        
      438 +```                                                           
      439 +                                                              
      440 +**This process-level liveness is explicitly NOT evidence that 
          +any job was                                                   
      441 +consumed or completed — there is no job to consume.** `worker`
          +'s `dev`                                                      
      442 +command (`tsx watch`) keeps its OS process alive by virtue of 
          +being a                                                       
      443 +*file-watching supervisor*, not because it is waiting on or pr
          +ocessing                                                      
      444 +work; `start` mode runs the identical application logic to com
          +pletion and                                                   
      445 +exits. Neither mode polls, subscribes to, or drains any queue,
          + in either                                                    
      446 +mode, today. Treat `ps`/`logs` output here the same way you'd 
          +treat                                                         
      447 +"the lights are on" — informative about the process, silent ab
          +out work.                                                     
      448 +                                                              
      449 +- **The background-job technology (`pg-boss` vs. `BullMQ`) rem
          +ains                                                          
      450 +  undecided**, per `context/stack.yml`'s `background_jobs.stat
          +us:                                                           
      451 +  undecided`, and this sub-issue does not select or implement 
          +one (out of                                                   
      452 +  scope by the task's own instruction).                       
      453 +- **`pg-boss@^12.26.3` is a declared dependency of `apps/worke
          +r`                                                            
      454 +  (`apps/worker/package.json`) but is imported and used nowher
          +e in                                                          
      455 +  `apps/worker/src`** — confirmed by searching the source tree
          + during                                                       
      456 +  this task (zero matches outside `package.json` itself and ge
          +nerated                                                       
      457 +  `dist/`/`node_modules/` output). A dependency's presence in 
      458 +  `package.json` is not evidence of an implemented queue integ
          +ration; see                                                   
      459 +  `docs/architecture/local-stack-inventory.md` §3.3/§8 for the
          + identical                                                    
      460 +  finding, reached independently during the preceding sub-issu
          +e.                                                            
      461 +- **No dashboard URL is documented here because none exists to
          + document.**                                                  
      462 +  Reporting this absence honestly, rather than inventing a URL
          + or                                                           
      463 +  building a dashboard to make this section look complete, is 
          +this                                                          
      464 +  sub-issue's explicit instruction.                           
      465 +- **Development-only concern, not a product feature** — once a
          + real                                                         
      466 +  queue consumer and (optionally) a dashboard are authorized a
          +nd                                                            
      467 +  implemented in a future, separately-scoped task, this sectio
          +n should be                                                   
      468 +  rewritten to describe them; until then, this honest "unavail
          +able" is                                                      
      469 +  the accurate state.                                         
      470 +                                                              
      471  ## Clean-checkout startup
      472  
      473  Each of `docker/web/Dockerfile`, `docker/api/Dockerfile`, and

● Now let's add the Diagnostics section, right after "Known limitations" and
  before "Tests".

  Read 1 file, ran 1 shell command (ctrl+o to expand)

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 128 lines
      819    logic specifically (as opposed to Docker's own escalation to
            `SIGKILL`
      820    after its stop grace period) was not individually isolated o
           r timed.
      821  
      822 +## Diagnostics                                                
      823 +                                                              
      824 +Implements local sub-issue [02.05](../../../context/notion-tas
          +ks/sub-issues/02-05.md).                                      
      825 +Actionable steps for the failure scenarios this sub-issue call
          +s out,                                                        
      826 +each with a real symptom (several captured from this document'
          +s own                                                         
      827 +real verification pass — see the completion report) and a scop
          +ed fix.                                                       
      828 +**None of these recommends restarting Docker Desktop, deleting
      829 +resources, or a broad prune as a routine step** — if a scoped 
          +fix below                                                     
      830 +doesn't resolve it, that is a blocker to report, not a reason 
          +to reach                                                      
      831 +for `docker system prune`.                                    
      832 +                                                              
      833 +### Docker is unavailable                                     
      834 +                                                              
      835 +**Symptom:** `docker compose up` (or any `docker` command) han
          +gs or fails                                                   
      836 +immediately with something like `error during connect` or `the
          + docker                                                       
      837 +daemon is not running`.                                       
      838 +                                                              
      839 +```bash                                                       
      840 +docker info --format "{{.ServerVersion}}"   # succeeds (prints
          + a version) only if the daemon is actually reachable          
      841 +```                                                           
      842 +                                                              
      843 +If this fails: start Docker Desktop (or your Docker daemon) **
          +yourself** —                                                  
      844 +this is a manual step, never an automated one. Every integrati
          +on test in                                                    
      845 +`tests/integration/docker-compose/` detects this the same way 
      846 +(`isDockerAvailableSync`) and reports a **skip with an explici
          +t blocker                                                     
      847 +message**, never a silent pass — the same honesty applies here
          +: report                                                      
      848 +"Docker is unavailable," don't claim the stack was verified.  
      849 +                                                              
      850 +### A host port is already occupied                           
      851 +                                                              
      852 +**Symptom** (real text captured during this sub-issue's own ve
          +rification,                                                   
      853 +starting a second project while the first still held the same 
          +port):                                                        
      854 +                                                              
      855 +```                                                           
      856 +Error response from daemon: failed to set up container network
          +ing: driver                                                   
      857 +failed programming external connectivity on endpoint ...: Bind
          + for                                                          
      858 +127.0.0.1:58634 failed: port is already allocated             
      859 +```                                                           
      860 +                                                              
      861 +**Fix:** either stop whatever already holds that port, or chan
          +ge the                                                        
      862 +conflicting variable in your `.env` — `NGINX_HTTP_PORT`, `MAIL
          +HOG_UI_PORT`,                                                 
      863 +or `SEAWEEDFS_S3_PORT` (see "Compose interpolation vs. applica
          +tion                                                          
      864 +environment loading" above for why editing `.env` is the right
          + lever                                                        
      865 +here, specifically) — then `docker compose up -d` again. To se
          +e what's                                                      
      866 +already bound before changing anything: `docker compose -p rel
          +is -f                                                         
      867 +docker-compose.yml ps` (another one of YOUR Compose projects) 
          +or your OS's                                                  
      868 +own port-listing tool (e.g. `netstat -ano | findstr :8080` on 
          +Windows) for                                                  
      869 +a non-Docker process.                                         
      870 +                                                              
      871 +### An image build fails                                      
      872 +                                                              
      873 +**Symptom:** `docker compose up --build` exits non-zero during
          + the `api`,                                                   
      874 +`web`, or `worker` image build step (`pnpm install --frozen-lo
          +ckfile` or                                                    
      875 +the `@relis/config` build failing inside the Dockerfile).     
      876 +                                                              
      877 +```bash                                                       
      878 +docker compose -p relis -f docker-compose.yml build api     # 
          +rebuild just one service, full output                         
      879 +docker compose -p relis -f docker-compose.yml build --no-cache
          + api   # rule out a stale cached layer specifically           
      880 +```                                                           
      881 +                                                              
      882 +Common causes: no network access to the pnpm/npm registry from
          + inside                                                       
      883 +the build (registry/proxy/firewall issue on the host), or a ge
          +nuinely                                                       
      884 +broken `pnpm-lock.yaml` (should not happen on an unmodified ch
          +eckout —                                                      
      885 +`pnpm install --frozen-lockfile` fails loudly rather than sile
          +ntly                                                          
      886 +resolving different versions if the lockfile and `package.json
          +` disagree).                                                  
      887 +                                                              
      888 +### Compose configuration itself is rejected                  
      889 +                                                              
      890 +**Symptom** (real text captured during this sub-issue's own ve
          +rification,                                                   
      891 +deliberately passing a malformed `NGINX_HTTP_PORT`):          
      892 +                                                              
      893 +```bash                                                       
      894 +$ NGINX_HTTP_PORT=not-a-number docker compose -f docker-compos
          +e.yml config --quiet                                          
      895 +invalid hostPort: not-a-number                                
      896 +```                                                           
      897 +                                                              
      898 +This fails **before** anything starts — `compose-config.test.t
          +s` asserts                                                    
      899 +the healthy case prints nothing on success; a non-empty messag
          +e (as                                                         
      900 +above) or a non-zero exit from `docker compose config --quiet`
          + means the                                                    
      901 +resolved configuration itself is invalid. Check whatever `.env
          +` value you                                                   
      902 +most recently changed first; this is a Compose-level failure, 
          +distinct                                                      
      903 +from an application's own `CONFIG_INVALID` diagnostic (next). 
      904 +                                                              
      905 +### A migration fails                                         
      906 +                                                              
      907 +**Symptom:** `migrate-control` or `migrate-project` exits non-
          +zero                                                          
      908 +instead of `0`.                                               
      909 +                                                              
      910 +```bash                                                       
      911 +docker compose -p relis -f docker-compose.yml logs migrate-con
          +trol                                                          
      912 +docker inspect <container-id> --format '{{.State.ExitCode}}'  
      913 +```                                                           
      914 +                                                              
      915 +Two distinct causes this stack's own tests already distinguish
          + — check                                                      
      916 +which one your log output matches before assuming either:     
      917 +                                                              
      918 +- **Missing/invalid configuration**, caught BEFORE Prisma ever
          + runs — a                                                     
      919 +  safe `CONFIG_INVALID` diagnostic naming only the affected va
          +riable                                                        
      920 +  category, never a value (`config-validation.test.ts`'s exact
          + scenario).                                                   
      921 +  Fix: correct the named variable in your `.env` (remembering 
          +it only                                                       
      922 +  affects this one-shot container's own `environment:` block —
          + see                                                          
      923 +  "Compose interpolation vs. application environment loading" 
          +above).                                                       
      924 +- **A valid configuration, unreachable target** — Prisma's own
          + real                                                         
      925 +  error, e.g. `` Error: P1001: Can't reach database server at 
      926 +  `postgres:5432` `` (`dependency-unavailable.test.ts`'s exact
          + scenario;                                                    
      927 +  this does echo the host/port/database name, but never the us
          +ername or                                                     
      928 +  password — see "Known limitations"). Fix: confirm `postgres`
          + itself is                                                    
      929 +  healthy first (`docker compose ps postgres`) — if it isn't, 
          +diagnose                                                      
      930 +  `postgres` itself before retrying the migration.            
      931 +                                                              
      932 +### The API is unavailable through the proxy                  
      933 +                                                              
      934 +**Symptom:** `/api/health`/`/api/ready` through nginx return a
          + `5xx`                                                        
      935 +(observed: `504 Gateway Timeout`, not an instant `502` — see "
          +Health and                                                    
      936 +readiness" above for exactly why) instead of `200`.           
      937 +                                                              
      938 +```bash                                                       
      939 +docker compose -p relis -f docker-compose.yml ps api     # is 
          +it even running, and healthy?                                 
      940 +docker compose -p relis -f docker-compose.yml logs api   # why
          + did it stop, or never start?                                 
      941 +docker compose -p relis -f docker-compose.yml restart api     
      942 +```                                                           
      943 +                                                              
      944 +This is specific to the `/api/` route — `/` (the web app) and 
      945 +`/nginx-health` keep working throughout (proxy liveness is ind
          +ependent of                                                   
      946 +API readiness; see "Health and readiness" above), so check tho
          +se two                                                        
      947 +first to confirm the failure is actually scoped to `api` and n
          +ot the                                                        
      948 +whole proxy/stack.                                            
      949 +                                                              
      950  ## Tests
      951  
      952  `tests/integration/docker-compose/`:

● Now let's update the Tests section's stack-smoke.test.ts bullet to mention the
  strengthened MailHog check.

  Read 1 file, ran 1 shell command (ctrl+o to expand)

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 5 lines, removed 1 line
      979    - `apps/worker`'s container is actually `running` (the `tsx 
           watch`
      980      supervisor's liveness) and its startup log line is present
            —
      981      explicitly not treated as evidence of job processing.
      982 -  - MailHog's UI reachable; a real signed SeaweedFS S3 PUT + G
          -ET                                                            
      982 +  - MailHog's root page and message-listing API are genuinely 
          +MailHog's                                                     
      983 +    OWN interface (local sub-issue 02.05) — its distinctive   
      984 +    `<title>MailHog</title>`/`ng-app="mailhogApp"` markup and 
          +exact                                                         
      985 +    `{total,count,start,items}` API shape, not merely an arbit
          +rary `200`                                                    
      986 +    (see "Mail capture" above); a real signed SeaweedFS S3 PUT
          + + GET                                                        
      987      round-trip.
      988    - `NGINX_HTTP_PORT` is chosen as a REAL free port with `getF
           reePort()`
      989      **before** `docker compose up` runs — not the host-port-`0
           `

● Now let's update README.md's Docker Compose section to link to the new
  documentation sections.

● Update(README.md)
  ⎿  Added 25 lines, removed 1 line
      370  seeds or backups) are documented in
      371  [docs/architecture/docker-compose-stack.md](docs/architecture/
           docker-compose-stack.md).
      372  Read that before relying on this stack for anything beyond loc
           al
      373 -iteration.                                                    
      373 +iteration. In particular, that document also covers (sub-issue
          + 02.05):                                                      
      374  
      375 +- **["Local access quick reference"](docs/architecture/docker-
          +compose-stack.md#local-access-quick-reference)**              
      376 +  — every host-facing URL and internal-only address in one tab
          +le, with                                                      
      377 +  its purpose and limitation.                                 
      378 +- **["Compose interpolation vs. application environment       
      379 +  loading"](docs/architecture/docker-compose-stack.md#compose-
          +interpolation-vs-application-environment-loading--two-separate
          +-mechanisms)**                                                
      380 +  — why editing the root `.env` does **not** reach `apps/api`/
          +`apps/web`/                                                   
      381 +  `apps/worker`'s own `loadEnvFiles` inside a container (it on
          +ly feeds                                                      
      382 +  Compose's own `${VAR}` interpolation in `docker-compose.yml`
          +).                                                            
      383 +- **["Diagnostics"](docs/architecture/docker-compose-stack.md#
          +diagnostics)**                                                
      384 +  — actionable steps (with real captured error text) for Docke
          +r being                                                       
      385 +  unavailable, an occupied host port, a failed image build, re
          +jected                                                        
      386 +  Compose configuration, a failed migration, and an unavailabl
          +e API                                                         
      387 +  upstream. None of it recommends restarting Docker Desktop or
          + a broad                                                      
      388 +  prune as a routine step.                                    
      389 +- **["Mail capture"](docs/architecture/docker-compose-stack.md
          +#mail-capture)**                                              
      390 +  — MailHog's UI/API, verified to genuinely be MailHog's own i
          +nterface                                                      
      391 +  (not an arbitrary `200`), and why no application mail adapte
          +r exists to                                                   
      392 +  send anything through it yet.                               
      393 +- **["Job observability"](docs/architecture/docker-compose-sta
          +ck.md#job-observability)**                                    
      394 +  — stated plainly: **no job-observability endpoint exists** i
          +n this                                                        
      395 +  stack. Only `apps/worker`'s container status/logs are availa
          +ble, and                                                      
      396 +  that liveness is explicitly **not** evidence any job was con
          +sumed or                                                      
      397 +  completed — there is no queue consumer to observe.          
      398 +                                                              
      399  `pnpm run test:integration` exercises the stack with real, dis
           posable
      400  Compose projects (`tests/integration/docker-compose/`) — inclu
           ding the
      401  persistence/reset/recreation lifecycle above and read-only Git
            checks

  Ran 1 shell command (ctrl+o to expand)

● Each heading appears exactly once — anchors are unambiguous. Now let's
  typecheck everything once more and run the strengthened test suite plus a full
  regression pass.

  Ran 1 shell command (ctrl+o to expand)

● Clean. Now the full integration regression suite (real Docker) to make sure
  the stack-smoke.test.ts change and everything else still passes together.

  Ran 1 shell command (ctrl+o to expand)

● All 25 files pass (324 passed, 8 expected skips, 0 failures). Let's check for
  leaked Docker resources and confirm the overall git status.

  Ran 1 shell command (ctrl+o to expand)

● No leaked Docker resources. The apps/web/tsconfig.json side-effect reappeared
  again (same known environment quirk from prior sessions) — reverting it since
  it's unrelated to this documentation task.

  Ran 2 shell commands (ctrl+o to expand)
● Let's add a consolidated "scoped commands" subsection at the top of
  Diagnostics, including port inspection which isn't yet documented as a
  developer-facing command.

  Read 1 file, ran 1 shell command (ctrl+o to expand)

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 34 lines
      830  doesn't resolve it, that is a blocker to report, not a reason 
           to reach
      831  for `docker system prune`.
      832  
      833 +### Scoped inspection commands (service status, logs, migratio
          +n exit codes, health/readiness, published ports)              
      834 +                                                              
      835 +Every command below is scoped to this ONE project (`-p relis`,
          + or                                                           
      836 +substitute your own `-p` if you started with a different one) 
          +and is                                                        
      837 +read-only — none of them changes any state, so they're always 
          +safe to                                                       
      838 +run first, before anything in the scenario-specific steps belo
          +w.                                                            
      839 +                                                              
      840 +```bash                                                       
      841 +# Service status at a glance (STATUS column shows "(healthy)",
          + "(health: starting)",                                        
      842 +# "Exited (0)" for a successful one-shot migration, or no heal
          +th annotation at all                                          
      843 +# for worker/mailhog/migrate-* — see "What a successful startu
          +p actually looks like").                                      
      844 +docker compose -p relis -f docker-compose.yml ps        # runn
          +ing services only                                             
      845 +docker compose -p relis -f docker-compose.yml ps -a     # incl
          +ude exited ones (migrate-control/migrate-project)             
      846 +                                                              
      847 +# Logs for one service (add -f to follow).                    
      848 +docker compose -p relis -f docker-compose.yml logs <service>  
      849 +                                                              
      850 +# A migration's real exit code — not "a database with the righ
          +t name exists"                                                
      851 +# (docker/postgres/init/ creates both databases unconditionall
          +y regardless of                                               
      852 +# whether either migration container ever runs — see "Database
          + isolation").                                                 
      853 +docker inspect $(docker compose -p relis -f docker-compose.yml
          + ps -a -q migrate-control) --format '{{.State.ExitCode}}'     
      854 +                                                              
      855 +# Health/readiness through the proxy (see "Health and readines
          +s" above for what each actually means).                       
      856 +curl http://localhost:${NGINX_HTTP_PORT:-8080}/nginx-health   
      857 +curl http://localhost:${NGINX_HTTP_PORT:-8080}/api/health     
      858 +curl http://localhost:${NGINX_HTTP_PORT:-8080}/api/ready      
      859 +                                                              
      860 +# What host port (if any) a service is ACTUALLY published on r
          +ight now.                                                     
      861 +docker compose -p relis -f docker-compose.yml port nginx 80   
          +  # -> 0.0.0.0:<port> or 127.0.0.1:<port>, per that service's 
          +binding                                                       
      862 +# The more complete picture (every declared container port, in
          +cluding ones NOT                                              
      863 +# published at all) — what port-exposure.test.ts itself assert
          +s on:                                                         
      864 +docker inspect <container-id> --format '{{json .HostConfig.Por
          +tBindings}}'                                                  
      865 +```                                                           
      866 +                                                              
      867  ### Docker is unavailable
      868  
      869  **Symptom:** `docker compose up` (or any `docker` command) han
           gs or fails

  Ran 2 shell commands (ctrl+o to expand)

● Let's do a final proofread of the key new sections.

  Read 1 file (ctrl+o to expand)

● This all reads well and is internally consistent. Let's do a final check of
  the git diff stat and wrap up.

  Ran 1 shell command (ctrl+o to expand)

● Summary — sub-issue 02.05 complete

  Documentation (primary deliverable) — extended existing docs, no competing 
  guide created

  relis/docs/architecture/docker-compose-stack.md — added, reconciled with
  existing content:
  - Prerequisites: split host-tool requirements (Docker-only vs. tests/native),
    versions pulled from package.json (engines.node: >=22.15, packageManager: 
    pnpm@11.17.0) and each docker/*/Dockerfile's FROM node:22-bookworm-slim.
  - "Compose interpolation vs. application environment loading": documented the
    gap I found by inspecting .dockerignore (excludes .env*) and
    docker-compose.yml (no env_file:, no bind-mount of root .env) — the root
    .env only feeds Compose's own ${VAR} interpolation; it never reaches
    loadEnvFiles inside a container.
  - Startup: expected docker compose ps output, the three "no health column"
    shapes (worker/mailhog/migrate-*) and why each is correct, build-time
    guidance without a fixed duration.
  - "Local access quick reference": one table — URL, host-facing/internal,
    purpose, limitation.
  - "Mail capture": MailHog identity verification (not an arbitrary 200),
    explicit "no application adapter" boundary, and why this task doesn't cross
    the SMTP internal-only boundary.
  - "Job observability": states plainly no endpoint exists (verified:
    apps/worker/src/main.ts opens no listener; no healthcheck:/port anywhere),
    only ps/logs are available, liveness ≠ job processing, pg-boss is declared
    but unused, queue tech stays undecided.
  - "Diagnostics": scoped inspection commands (status/logs/migration exit
    code/health/port inspection) + 6 actionable failure scenarios, two with real
    captured error text (port conflict, rejected config) from this task's own
    verification run.

  relis/README.md — linked the new sections from the existing "Docker Compose
  (local stack)" overview.

  Verification — real execution, not inspection alone

  - Automated (real Docker, guarded disposable projects already in place from
    02.03/02.04):
    - Strengthened stack-smoke.test.ts's MailHog check to verify real identity
      (<title>MailHog</title>, ng-app="mailhogApp", exact
      {total,count,start,items} API shape) instead of a bare 200.
    - pnpm run typecheck → pass. npx vitest run --config 
      vitest.integration.config.ts (full suite) → 25 files, 324 passed, 8 
      skipped (expected), 0 failed.
  - Manual, guarded walkthrough (uniquely-named disposable projects
    relis-docverify-*, ephemeral/free ports, volume ownership confirmed by label
    before the destructive teardown): real docker compose up --build, ps,
    docker inspect exit codes, proxy health/ready curls, MailHog UI/API,
    SeaweedFS health convergence, worker logs, a real port-conflict error, and a
    real rejected-config error. Scoped down --volumes --remove-orphans
    afterward; docker ps/volume/network ls confirmed empty both times.

  Limitations / honesty notes

  - Not a from-scratch clean checkout: Docker's build-layer cache was warm from
    earlier sub-issues' sessions (same package.json/pnpm-lock.yaml), so this did
    not prove a truly cache-cold build; I did not run docker builder 
    prune/--no-cache to force one, since that wasn't requested and is expensive.
    Stated explicitly in the docs' build-time guidance.
  - Did not attempt a live SMTP round-trip into MailHog (internal-only port,
    deliberately out of this task's scope) — identity verification uses
    MailHog's own UI/API markers instead; documented why.
  - No job dashboard/queue consumer was implemented or invented — reported as
    unavailable per instructions.

  Changed files (no unrelated changes)

  README.md, docs/architecture/docker-compose-stack.md,
  tests/integration/docker-compose/stack-smoke.test.ts. (An incidental Next.js
  tooling side-effect touched apps/web/tsconfig.json twice during this session —
  reverted both times via git checkout; final git status shows none of it.)

  Not closing the parent issue, not starting another sub-issue, no commit, no
  push, per instructions.

✻ Cooked for 21m 51s · done 17:23

※ recap: Sub-issue 02.05 (docs for startup, diagnostics, MailHog, and job observability) is done, including real Docker verification. Next action: await your review before closing anything or starting 02.06 or beyond. (disable recaps in /config)

❯ Fix the two documentation findings in sub-issue 02.05. Do not change          
  application code or expand the scope.                                         
                                                                                
  1. Proxy diagnostic commands                                                  
     The documented ${NGINX_HTTP_PORT:-8080} shell expression does not read     
  Compose's .env file. If the port is changed only in .env, the curl commands   
  may target the wrong endpoint.                                                
  - Document discovering the actual published address with docker compose -p    
  <project> -f docker-compose.yml port nginx 80.                                
  - Use that address for the health/readiness probes.                           
  - Provide clearly labeled, copyable commands for Bash and PowerShell without  
  assuming .env variables are exported.                                         
  - Correct repeated instances of the same assumption in the affected           
  documentation.                                                                
                                                                                
  2. Migration configuration troubleshooting                                    
     The Compose migration services construct their database URLs from          
  POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB, and PROJECT_TEST_DB_NAME. They 
  do not directly consume the root .env's CONTROL_DATABASE_URL or               
  PROJECT_DATABASE_URL.                                                         
  - Explain exactly which settings to inspect and correct for each Compose      
  migration target.                                                             
  - Distinguish this from native migration execution.                           
  - Preserve the warning that changing initialization settings does not update  
  an existing PostgreSQL volume.                                                
  - Do not suggest destructive reset as the default fix or print credentials in 
  diagnostic examples.                                                          
                                                                                
  Verify the instructions against the actual Compose configuration. Check the   
  port-discovery workflow using a non-default port, if safe disposable          
  infrastructure is available; otherwise report that execution limitation.      
  Preserve existing guarded provisioning and cleanup.                           
                                                                                
  Keep edits limited to the existing documentation. Report changed sections and 
  checks actually performed. Do not commit, push, or start another sub-issue.   

  Read 1 file, ran 3 shell commands (ctrl+o to expand)

● Let's re-verify the exact Compose migration service definitions against the
  current file before writing the fix.

  Ran 1 shell command (ctrl+o to expand)

● Confirmed exactly as recalled. Now let's verify the volume-reinit warning's
  exact wording in the existing doc to reuse it precisely.

  Read 1 file, ran 1 shell command (ctrl+o to expand)

● Now let's verify the port-discovery workflow for real, using a non-default
  port on a guarded disposable project.

  Ran 2 shell commands (ctrl+o to expand)

● Excellent — bug confirmed exactly as described. Now let's verify the
  PowerShell equivalent against the same live container.

  Ran 1 shell command (ctrl+o to expand)

● PowerShell flow verified too. Now let's tear down this guarded verification
  project.

  Ran 1 shell command (ctrl+o to expand)

● Now let's write the documentation fixes. First, the "What a successful startup
  actually looks like" section's probe commands.

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 43 lines, removed 6 lines
      200    for a different reason: they are not long-running services t
           o probe —
      201    success is their own process exiting `0`.
      202  
      203 -Through the proxy, immediately after `nginx` itself reports `(
          -healthy)`:                                                    
      203 +Through the proxy, immediately after `nginx` itself reports `(
          +healthy)`.                                                    
      204 +**Discover the actual published address first — never assume  
      205 +`localhost:8080`.** `NGINX_HTTP_PORT:-8080` is a default writt
          +en directly                                                   
      206 +inside `docker-compose.yml`; it is resolved by `docker compose
          +` itself                                                      
      207 +from the root `.env` file (see "Compose interpolation vs. appl
          +ication                                                       
      208 +environment loading" above), which your shell has **no visibil
          +ity into                                                      
      209 +at all** unless you've explicitly exported it there yourself. 
          +A shell                                                       
      210 +expression like `${NGINX_HTTP_PORT:-8080}` in your OWN termina
          +l reads                                                       
      211 +your shell's environment, not `.env` — if `.env` sets a differ
          +ent port                                                      
      212 +and that variable was never exported into your shell, the expr
          +ession                                                        
      213 +silently falls back to the literal `8080`, which may not be pu
          +blished at                                                    
      214 +all (**verified by real execution**: with `NGINX_HTTP_PORT=192
          +34` set                                                       
      215 +only in `.env`, a fresh shell's `${NGINX_HTTP_PORT:-8080}` sti
          +ll resolved                                                   
      216 +to `8080`, and a request to that address failed to connect — n
          +othing was                                                    
      217 +listening there). Ask Docker for the real, currently-published
          + address                                                      
      218 +instead, then use exactly that in every probe below:          
      219  
      220 +**Bash:**                                                     
      221 +                                                              
      222  ```bash
      206 -curl http://localhost:8080/nginx-health   # -> 200, body "ok" 
          - (proxy liveness only)                                        
      207 -curl http://localhost:8080/api/health     # -> 200, {"status":
          -"ok","message":"...","service":"api"}                         
      208 -curl http://localhost:8080/api/ready      # -> 200, {"status":
          -"ok","service":"api","checks":{"network":"ok","cors":"ok"}}   
      223 +NGINX_ADDR=$(docker compose -p relis -f docker-compose.yml por
          +t nginx 80)                                                   
      224 +echo "$NGINX_ADDR"   # e.g. 127.0.0.1:19234 — the REAL address
          +, whatever NGINX_HTTP_PORT actually resolved to               
      225 +                                                              
      226 +curl "http://${NGINX_ADDR}/nginx-health"   # -> 200, body "ok"
          +  (proxy liveness only)                                       
      227 +curl "http://${NGINX_ADDR}/api/health"     # -> 200, {"status"
          +:"ok","message":"...","service":"api"}                        
      228 +curl "http://${NGINX_ADDR}/api/ready"      # -> 200, {"status"
          +:"ok","service":"api","checks":{"network":"ok","cors":"ok"}}  
      229  ```
      230  
      211 -(Substitute your own `${NGINX_HTTP_PORT:-8080}` if you changed
          - it.) Open                                                    
      212 -`http://localhost:8080/` in a browser for the application itse
          -lf.                                                           
      231 +**PowerShell:**                                               
      232  
      233 +```powershell                                                 
      234 +$NginxAddr = docker compose -p relis -f docker-compose.yml por
          +t nginx 80                                                    
      235 +Write-Host $NginxAddr   # e.g. 127.0.0.1:19234                
      236 +                                                              
      237 +Invoke-WebRequest "http://$NginxAddr/nginx-health" -UseBasicPa
          +rsing                                                         
      238 +Invoke-WebRequest "http://$NginxAddr/api/health" -UseBasicPars
          +ing                                                           
      239 +Invoke-WebRequest "http://$NginxAddr/api/ready" -UseBasicParsi
          +ng                                                            
      240 +```                                                           
      241 +                                                              
      242 +Both forms were run against a real, disposable, guarded projec
          +t with a                                                      
      243 +deliberately non-default `NGINX_HTTP_PORT` (`19234`, chosen fa
          +r from the                                                    
      244 +default specifically to rule out a coincidental match) during 
          +this                                                          
      245 +sub-issue's own verification pass: `docker compose port nginx 
          +80` printed                                                   
      246 +`127.0.0.1:19234`, and every probe against that discovered add
          +ress                                                          
      247 +returned `200` in both Bash and PowerShell. Open the same disc
          +overed                                                        
      248 +address in a browser (`http://<discovered-address>/`) for the 
          +application                                                   
      249 +itself.                                                       
      250 +                                                              
      251  ## Shutdown and reset
      252  
      253  ```bash

● Now let's fix the "Scoped inspection commands" block in Diagnostics — reorder
  to discover the port first, and add PowerShell equivalents.

  Read 1 file, ran 1 shell command (ctrl+o to expand)

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 47 lines, removed 6 lines
      874  read-only — none of them changes any state, so they're always 
           safe to
      875  run first, before anything in the scenario-specific steps belo
           w.
      876  
      877 +**Discover the published port BEFORE probing health/readiness 
          +— never                                                       
      878 +assume `localhost:8080`.** The health/readiness probes below d
          +epend on                                                      
      879 +knowing the real address first, which is exactly why the port-
          +inspection                                                    
      880 +step comes before them here (not after, as an earlier draft of
          + this                                                         
      881 +document had it): `${NGINX_HTTP_PORT:-8080}` typed directly in
          +to your own                                                   
      882 +shell reads your shell's environment, never the root `.env` fi
          +le that                                                       
      883 +`docker compose` itself resolves it from (see "Compose interpo
          +lation vs.                                                    
      884 +application environment loading" above) — if that variable isn
          +'t                                                            
      885 +separately exported into your shell, the expression silently f
          +alls back                                                     
      886 +to `8080`, which may not be published at all. **Verified by re
          +al                                                            
      887 +execution** against a disposable, guarded project with a delib
          +erately                                                       
      888 +non-default `NGINX_HTTP_PORT` (`19234`): `docker compose port 
          +nginx 80`                                                     
      889 +printed the real `127.0.0.1:19234`, while a naive, unexported 
      890 +`${NGINX_HTTP_PORT:-8080}` in a fresh shell still resolved to 
          +`8080` and                                                    
      891 +failed to connect.                                            
      892 +                                                              
      893 +**Bash:**                                                     
      894 +                                                              
      895  ```bash
      896  # Service status at a glance (STATUS column shows "(healthy)",
            "(health: starting)",
      897  # "Exited (0)" for a successful one-shot migration, or no heal
           th annotation at all
     ...
      907  # whether either migration container ever runs — see "Database
            isolation").
      908  docker inspect $(docker compose -p relis -f docker-compose.yml
            ps -a -q migrate-control) --format '{{.State.ExitCode}}'
      909  
      892 -# Health/readiness through the proxy (see "Health and readines
          -s" above for what each actually means).                       
      893 -curl http://localhost:${NGINX_HTTP_PORT:-8080}/nginx-health   
      894 -curl http://localhost:${NGINX_HTTP_PORT:-8080}/api/health     
      895 -curl http://localhost:${NGINX_HTTP_PORT:-8080}/api/ready      
      910 +# What host port (if any) a service is ACTUALLY published on r
          +ight now —                                                    
      911 +# discover this FIRST, then use it below.                     
      912 +NGINX_ADDR=$(docker compose -p relis -f docker-compose.yml por
          +t nginx 80)                                                   
      913 +echo "$NGINX_ADDR"   # e.g. 127.0.0.1:19234 — the REAL address
          +, whatever NGINX_HTTP_PORT actually resolved to               
      914  
      897 -# What host port (if any) a service is ACTUALLY published on r
          -ight now.                                                     
      898 -docker compose -p relis -f docker-compose.yml port nginx 80   
          -  # -> 0.0.0.0:<port> or 127.0.0.1:<port>, per that service's 
          -binding                                                       
      915 +# Health/readiness through the proxy, using the DISCOVERED add
          +ress (see "Health and readiness" above for what each actually 
          +means).                                                       
      916 +curl "http://${NGINX_ADDR}/nginx-health"                      
      917 +curl "http://${NGINX_ADDR}/api/health"                        
      918 +curl "http://${NGINX_ADDR}/api/ready"                         
      919 +                                                              
      920  # The more complete picture (every declared container port, in
           cluding ones NOT
      921  # published at all) — what port-exposure.test.ts itself assert
           s on:
      922  docker inspect <container-id> --format '{{json .HostConfig.Por
           tBindings}}'
      923  ```
      924  
      925 +**PowerShell:**                                               
      926 +                                                              
      927 +```powershell                                                 
      928 +# Service status, logs, and migration exit code — identical co
          +mmands, no shell-specific syntax.                             
      929 +docker compose -p relis -f docker-compose.yml ps              
      930 +docker compose -p relis -f docker-compose.yml ps -a           
      931 +docker compose -p relis -f docker-compose.yml logs <service>  
      932 +docker inspect $(docker compose -p relis -f docker-compose.yml
          + ps -a -q migrate-control) --format '{{.State.ExitCode}}'     
      933 +                                                              
      934 +# Discover the real published address first, then use it.     
      935 +$NginxAddr = docker compose -p relis -f docker-compose.yml por
          +t nginx 80                                                    
      936 +Write-Host $NginxAddr   # e.g. 127.0.0.1:19234                
      937 +                                                              
      938 +Invoke-WebRequest "http://$NginxAddr/nginx-health" -UseBasicPa
          +rsing                                                         
      939 +Invoke-WebRequest "http://$NginxAddr/api/health" -UseBasicPars
          +ing                                                           
      940 +Invoke-WebRequest "http://$NginxAddr/api/ready" -UseBasicParsi
          +ng                                                            
      941 +                                                              
      942 +docker inspect <container-id> --format '{{json .HostConfig.Por
          +tBindings}}'                                                  
      943 +```                                                           
      944 +                                                              
      945  ### Docker is unavailable
      946  
      947  **Symptom:** `docker compose up` (or any `docker` command) han
           gs or fails

● Now let's fix the "occupied port" section's hardcoded :8080 netstat example to
  point back to the discovery workflow.

  Read 1 file, ran 1 shell command (ctrl+o to expand)

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 7 lines, removed 3 lines
      976  environment loading" above for why editing `.env` is the right
            lever
      977  here, specifically) — then `docker compose up -d` again. To se
           e what's
      978  already bound before changing anything: `docker compose -p rel
           is -f
      979 -docker-compose.yml ps` (another one of YOUR Compose projects) 
          -or your OS's                                                  
      980 -own port-listing tool (e.g. `netstat -ano | findstr :8080` on 
          -Windows) for                                                  
      981 -a non-Docker process.                                         
      979 +docker-compose.yml ps` (another one of YOUR Compose projects) 
          +— do not                                                      
      980 +assume the conflicting port is `8080`; read it from `.env`, or
          + discover                                                     
      981 +whatever is ACTUALLY published right now with `docker compose 
          +-p relis -f                                                   
      982 +docker-compose.yml port nginx 80` (see "Scoped inspection comm
          +ands"                                                         
      983 +above) — or your OS's own port-listing tool for a non-Docker p
          +rocess                                                        
      984 +(substitute the real port number, e.g. `netstat -ano | findstr
          + :<port>`                                                     
      985 +on Windows PowerShell/cmd).                                   
      986  
      987  ### An image build fails
      988  

● Now let's fix finding #2: the migration troubleshooting section.

  Read 1 file, ran 1 shell command (ctrl+o to expand)

● Now let's replace this section with the corrected, verified explanation.

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 77 lines, removed 3 lines
      1034  - **Missing/invalid configuration**, caught BEFORE Prisma eve
            r runs — a
      1035    safe `CONFIG_INVALID` diagnostic naming only the affected v
            ariable
      1036    category, never a value (`config-validation.test.ts`'s exac
            t scenario).
      1037 -  Fix: correct the named variable in your `.env` (remembering
           - it only                                                     
      1038 -  affects this one-shot container's own `environment:` block 
           -— see                                                        
      1039 -  "Compose interpolation vs. application environment loading"
           - above).                                                     
      1037 +  **Fix — inspect and correct the right SOURCE variables; see
           + "Which                                                      
      1038 +  settings actually apply" immediately below before touching 
           +`.env` —                                                     
      1039 +  `CONTROL_DATABASE_URL`/`PROJECT_DATABASE_URL` themselves ar
           +e NOT what                                                   
      1040 +  these two Compose services read `.env` for, and editing the
           +m there has                                                  
      1041 +  no effect on this failure.**                               
      1042  - **A valid configuration, unreachable target** — Prisma's ow
            n real
      1043    error, e.g. `` Error: P1001: Can't reach database server at
      1044    `postgres:5432` `` (`dependency-unavailable.test.ts`'s exac
            t scenario;
     ...
      1047    healthy first (`docker compose ps postgres`) — if it isn't,
             diagnose
      1048    `postgres` itself before retrying the migration.
      1049  
      1050 +#### Which settings actually apply, per target — Compose vs. 
           +native                                                       
      1051 +                                                             
      1052 +**Verified directly against `docker-compose.yml`'s own `migra
           +te-control`/                                                 
      1053 +`migrate-project` service definitions.** Each one's `environm
           +ent:` block                                                  
      1054 +sets `CONTROL_DATABASE_URL`/`PROJECT_DATABASE_URL` as a liter
           +al value                                                     
      1055 +**assembled by Compose interpolation from smaller `POSTGRES_*
           +` pieces** —                                                 
      1056 +it does **not** read a `CONTROL_DATABASE_URL`/`PROJECT_DATABA
           +SE_URL` line                                                 
      1057 +from `.env` directly, even though `.env.example` happens to d
           +efine lines                                                  
      1058 +with those exact same names (for a *different* purpose — see 
           +"Native                                                      
      1059 +migration," below). Shown here as a shape with the SOURCE var
           +iable names                                                  
      1060 +in place of any value, never a real connection string or cred
           +ential:                                                      
      1061 +                                                             
      1062 +| Compose target | URL variable the migration CODE validates 
           +| Built by Compose interpolation from (root `.env`, read by `
           +docker compose` itself only) | Host/port |                   
      1063 +| --- | --- | --- | --- |                                    
      1064 +| `migrate-control` | `CONTROL_DATABASE_URL` | `postgresql://
           +<POSTGRES_USER>:<POSTGRES_PASSWORD>@postgres:5432/<POSTGRES_D
           +B>` | Fixed at `postgres:5432` — not independently configurab
           +le here. |                                                   
      1065 +| `migrate-project` | `PROJECT_DATABASE_URL` | `postgresql://
           +<POSTGRES_USER>:<POSTGRES_PASSWORD>@postgres:5432/<PROJECT_TE
           +ST_DB_NAME>` | Same, fixed at `postgres:5432`. |             
      1066 +                                                             
      1067 +**So, to fix a Compose migration's configuration:** inspect a
           +nd correct                                                   
      1068 +`POSTGRES_USER`, `POSTGRES_PASSWORD`, and — depending on whic
           +h target                                                     
      1069 +failed — `POSTGRES_DB` (`migrate-control`) or `PROJECT_TEST_D
           +B_NAME`                                                      
      1070 +(`migrate-project`), all in the root `.env`. **Editing the   
      1071 +`CONTROL_DATABASE_URL`/`PROJECT_DATABASE_URL` lines in that S
           +AME `.env`                                                   
      1072 +file has no effect whatsoever on either Compose service** — n
           +either one                                                   
      1073 +is wired to read them (see "Compose interpolation vs. applica
           +tion                                                         
      1074 +environment loading" above for the general version of this di
           +stinction).                                                  
      1075 +                                                             
      1076 +**Native migration execution reads the opposite set of variab
           +les.**                                                       
      1077 +Running the identical migration OUTSIDE Docker —             
      1078 +`pnpm --filter @relis/database run migrate control` /        
      1079 +`run migrate project`, from `relis/` — uses `@relis/config`'s
           + own                                                         
      1080 +`loadEnvFiles` (documented in `README.md` "Environment loadin
           +g and                                                        
      1081 +override precedence") to read `CONTROL_DATABASE_URL`/        
      1082 +`PROJECT_DATABASE_URL` **directly, as a complete connection s
           +tring**, from                                                
      1083 +`.env`/`.env.local`/the real shell environment — it never ass
           +embles one                                                   
      1084 +from `POSTGRES_USER`/`POSTGRES_PASSWORD`/`POSTGRES_DB`/      
      1085 +`PROJECT_TEST_DB_NAME` at all. These are genuinely two differ
           +ent                                                          
      1086 +mechanisms sharing one `.env` file for two different sets of 
           +variables;                                                   
      1087 +a value correct for one migration path (native) is not what t
           +he other                                                     
      1088 +path (Compose) consumes, and vice versa.                     
      1089 +                                                             
      1090 +**⚠️ Changing `POSTGRES_DB` or `PROJECT_TEST_DB_NAME` to "fix
           +" a Compose                                                  
      1091 +migration does not retroactively apply to an already-existing
      1092 +`<project>_postgres-data` volume** — this is the SAME warning
           + already                                                     
      1093 +stated in "Honest limitations of 'recreated'" above, repeated
           + here                                                        
      1094 +because it is the single most common way this specific fix go
           +es wrong:                                                    
      1095 +`docker/postgres/init/` only runs on a genuinely first init, 
           +so renaming                                                  
      1096 +either variable against a volume that already exists does not
           + create a                                                    
      1097 +database under the new name — the migration will then fail di
           +fferently                                                    
      1098 +(the target database itself does not exist), not because the 
           +new name is                                                  
      1099 +wrong. **This is not a reason to reset by default.** First co
           +nfirm                                                        
      1100 +whether the database name you actually need already exists on
           + the                                                         
      1101 +running server (read-only, no credential in the example):    
      1102 +                                                             
      1103 +```bash                                                      
      1104 +docker compose -p relis -f docker-compose.yml exec -T postgre
           +s psql -U "$POSTGRES_USER" -d postgres -tAc "SELECT datname F
           +ROM pg_database;"                                            
      1105 +```                                                          
      1106 +                                                             
      1107 +If it's genuinely missing and you intend to keep the existing
           + volume's                                                    
      1108 +other data, create it manually (a non-destructive, explicit D
           +DL                                                           
      1109 +statement — substitute the real target name, never paste a pa
           +ssword into                                                  
      1110 +this command: `psql` picks up `PGPASSWORD` from the container
           +'s own                                                       
      1111 +already-configured environment, or prompts interactively):   
      1112 +                                                             
      1113 +```bash                                                      
      1114 +docker compose -p relis -f docker-compose.yml exec -T postgre
           +s psql -U "$POSTGRES_USER" -d postgres -c "CREATE DATABASE <t
           +arget-db-name>;"                                             
      1115 +```                                                          
      1116 +                                                             
      1117 +Only if you genuinely want a clean slate — never as the routi
           +ne or                                                        
      1118 +default response to this specific failure — does the explicit
           +, already                                                    
      1119 +project-scoped reset in ["Shutdown and reset"](#shutdown-and-
           +reset) apply                                                 
      1120 +instead.                                                     
      1121 +                                                             
      1122  ### The API is unavailable through the proxy
      1123  
      1124  **Symptom:** `/api/health`/`/api/ready` through nginx return 
            a `5xx`
● I caught an inconsistency — my own fix for finding #2 repeats the exact
  shell-expansion assumption from finding #1 ($POSTGRES_USER expanded by the
  host shell, not the container). Let me verify and correct this.

  Ran 3 shell commands (ctrl+o to expand)

● Verified. Now let's tear down this guarded verification project, then fix the
  two commands in the doc.

  Ran 1 shell command (ctrl+o to expand)

● Clean teardown confirmed. Now let's fix the two commands in the doc to expand
  $POSTGRES_USER inside the container.

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 2 lines, removed 2 lines
      1107  If it's genuinely missing and you intend to keep the existing
             volume's
      1108  other data, create it manually (a non-destructive, explicit D
            DL
      1109  statement — substitute the real target name, never paste a pa
            ssword into
      1110 -this command: `psql` picks up `PGPASSWORD` from the container
           -'s own                                                       
      1110 +this command: `psql` picks up its own credentials from the co
           +ntainer's                                                    
      1111  already-configured environment, or prompts interactively):
      1112  
      1113  ```bash
      1114 -docker compose -p relis -f docker-compose.yml exec -T postgre
           -s psql -U "$POSTGRES_USER" -d postgres -c "CREATE DATABASE <t
           -arget-db-name>;"                                             
      1114 +docker compose -p relis -f docker-compose.yml exec -T postgre
           +s sh -c 'psql -U "$POSTGRES_USER" -d postgres -c "CREATE DATA
           +BASE <target-db-name>;"'                                     
      1115  ```
      1116  
      1117  Only if you genuinely want a clean slate — never as the routi
            ne or

● Update(docs\architecture\docker-compose-stack.md)
  ⎿  Added 7 lines, removed 2 lines
      1098  (the target database itself does not exist), not because the 
            new name is
      1099  wrong. **This is not a reason to reset by default.** First co
            nfirm
      1100  whether the database name you actually need already exists on
             the
      1101 -running server (read-only, no credential in the example):    
      1101 +running server (read-only, no credential in the example — `$P
           +OSTGRES_USER`                                                
      1102 +here is expanded BY THE CONTAINER's own shell via `sh -c`, wh
           +ere it is                                                    
      1103 +genuinely set from the same `environment:` block Compose alre
           +ady                                                          
      1104 +resolved; expanding it from your own host shell instead would
           + silently                                                    
      1105 +read an unrelated or empty value, the identical class of mist
           +ake as the                                                   
      1106 +port-discovery issue above):                                 
      1107  
      1108  ```bash
      1104 -docker compose -p relis -f docker-compose.yml exec -T postgre
           -s psql -U "$POSTGRES_USER" -d postgres -tAc "SELECT datname F
           -ROM pg_database;"                                            
      1109 +docker compose -p relis -f docker-compose.yml exec -T postgre
           +s sh -c 'psql -U "$POSTGRES_USER" -d postgres -tAc "SELECT da
           +tname FROM pg_database;"'                                    
      1110  ```
      1111  
      1112  If it's genuinely missing and you intend to keep the existing
             volume's

  Ran 1 shell command (ctrl+o to expand)

● Both actual commands are now correctly wrapped. Let's do a full read-through
  of the final migration section for coherence.

  Read 1 file (ctrl+o to expand)

● This reads correctly and matches the verified behavior. Now let's check
  markdown fence balance and run a final sanity check on the whole doc.

  Ran 1 shell command (ctrl+o to expand)

● Even count, balanced. Now let's check the remaining ${NGINX_HTTP_PORT:-8080}
  instances to confirm they're all in descriptive tables (not copyable
  commands), which is the correct, acceptable usage.

  Ran 2 shell commands (ctrl+o to expand)

● All remaining curl commands now use the discovered $NGINX_ADDR variable,
  consistently. Let's also check the README.md doesn't repeat the same
  assumption anywhere.

  Ran 3 shell commands (ctrl+o to expand)

● Let's confirm the new #shutdown-and-reset anchor link resolves correctly and
  do a final typecheck sanity pass (docs-only change, but confirming nothing
  else broke).

  Ran 1 shell command (ctrl+o to expand)

● Anchor matches. Let's run the typecheck to confirm nothing else is affected.

  Ran 1 shell command (ctrl+o to expand)

● Clean. Let's do a final check for leftover Docker resources from this
  session's verification probes.

  Ran 1 shell command (ctrl+o to expand)

● All clean. Let's check for the apps/web/tsconfig.json environment side-effect
  one more time before reporting.

  Ran 1 shell command (ctrl+o to expand)

● No stray tsconfig.json side-effect this time. Only
  docs/architecture/docker-compose-stack.md was edited in this fix pass — the
  other two files are untouched carryovers from the prior turn's 02.05 work.

  Summary — both documentation findings fixed in 
  docs/architecture/docker-compose-stack.md

  Finding 1 — proxy diagnostic commands

  - "What a successful startup actually looks like": replaced the curl 
    http://localhost:8080/... block + "(substitute ${NGINX_HTTP_PORT:-8080} if
    changed)" aside with a discover-then-probe flow: docker compose -p relis -f 
    docker-compose.yml port nginx 80 captured into a variable, then used in the
    probes — given as labeled Bash and PowerShell versions.
  - "Scoped inspection commands": reordered so port discovery happens before the
    health/readiness curls (previously the port command came after, unused by
    them); added the PowerShell equivalent for the whole block.
  - "A host port is already occupied": removed the bare findstr :8080 example in
    favor of discovering the real port first, explicitly calling out not to
    assume 8080.
  - Added an explanatory note (reused at both sites) stating why
    ${NGINX_HTTP_PORT:-8080} typed into a shell doesn't see .env — it reads the
    shell's own environment, which .env never populates unless exported.

  Finding 2 — migration configuration troubleshooting

  - Replaced the incorrect "Fix: correct the named variable in your .env" (which
    pointed at CONTROL_DATABASE_URL/PROJECT_DATABASE_URL — variables the
    Compose migration services never read) with a new "Which settings actually 
    apply, per target — Compose vs. native" subsection:
    - Table mapping each Compose target to the real source variables
      (POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB for migrate-control; same
      plus PROJECT_TEST_DB_NAME for migrate-project), shown only as placeholder
      shapes, never real values/credentials.
    - Explicit distinction from native pnpm --filter @relis/database run migrate
      <target>, which reads CONTROL_DATABASE_URL/PROJECT_DATABASE_URL directly
      via loadEnvFiles.
    - Preserved (and cross-referenced) the existing "docker/postgres/init/ only
      runs on a genuinely first init" warning, applied specifically to this
      failure mode.
    - Default fix is non-destructive (inspect/create the missing database); the
      existing scoped reset is mentioned only as a last resort, not the default.
    - While verifying, caught and fixed my own draft repeating the same class of
      bug: $POSTGRES_USER is now expanded inside the container via sh -c, not
      the host shell.

  Verification performed (real Docker, guarded, no application code touched)

  - relis-portverify-* (nginx+web+api, non-default NGINX_HTTP_PORT=19234):
    confirmed docker compose port nginx 80 → 127.0.0.1:19234;
    curl/Invoke-WebRequest against that address → 200 for /nginx-health and
    /api/* in both Bash and PowerShell; confirmed the naive
    ${NGINX_HTTP_PORT:-8080} in a fresh shell falls back to 8080 and fails to
    connect.
  - relis-migverify-* (postgres only): confirmed the host-shell-expansion form
    fails (role "root" does not exist) and the sh -c-wrapped form succeeds,
    listing real databases; verified the CREATE DATABASE form.
  - Both projects: ownership confirmed by Compose label before the destructive
    down --volumes; docker ps/volume/network ls empty afterward — no leaked
    resources, no existing user stack touched.
  - pnpm run typecheck:tests → pass (doc-only change, confirms nothing else
    broke).

  No application code changed; no commit/push; no new sub-issue started.

✻ Cogitated for 7m 57s · done 17:40

※ recap: Building the ReLiS local Docker Compose dev stack. Just fixed two doc issues in sub-issue 02.05 (proxy diagnostic commands and migration troubleshooting) and verified them; next step is your review before moving on.