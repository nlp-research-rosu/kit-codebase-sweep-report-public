# Daytona stale public-preview authorization

This is a source-confirmed conditional P1. A stale public-status cache entry could preserve anonymous preview access after a sandbox became private.

The reviewed revision is [`4ee2c6365b851cbc7073ca4cea2f9ebba0caf832`](https://github.com/daytonaio/daytona/commit/4ee2c6365b851cbc7073ca4cea2f9ebba0caf832).

The evidence includes a pinned source trace and a constructed state-transition claim. The retained audit did not complete a full deployment reproduction.

## 1. System boundary

Daytona exposes browser-accessible previews for services running inside sandboxes. Public sandboxes allow anonymous preview traffic on ordinary ports. Private sandboxes require a preview credential or authenticated access. Reserved internal ports require authentication in either state.

The proxy cached each sandbox's public status to avoid an API request for every preview connection. That cached boolean became an authorization input because it determined whether the proxy called its authentication path.

Once the control plane committed `public=false`, an earlier cache value could no longer authorize anonymous preview access.

## 2. Root cause

The control-plane state change and the proxy's cached authorization state were updated through separate operations with different failure behavior.

The proxy's [`getSandboxPublic`](https://github.com/daytonaio/daytona/blob/4ee2c6365b851cbc7073ca4cea2f9ebba0caf832/apps/proxy/pkg/proxy/get_sandbox_target.go#L163-L200) returned a cached value before asking the API and stored fresh results for one hour. [`GetProxyTarget`](https://github.com/daytonaio/daytona/blob/4ee2c6365b851cbc7073ca4cea2f9ebba0caf832/apps/proxy/pkg/proxy/get_sandbox_target.go#L60-L78) skipped authentication for a cached-public sandbox on a non-reserved port.

The API tried to delete the proxy cache entry when public status changed. Its [invalidation service](https://github.com/daytonaio/daytona/blob/4ee2c6365b851cbc7073ca4cea2f9ebba0caf832/apps/api/src/sandbox/services/proxy-cache-invalidation.service.ts#L27-L50) caught deletion failures and logged them while allowing the privacy transition to complete.

The proxy also supported [Redis and in-process cache modes](https://github.com/daytonaio/daytona/blob/4ee2c6365b851cbc7073ca4cea2f9ebba0caf832/apps/proxy/pkg/proxy/proxy.go#L98-L128). API-side Redis deletion could not clear a proxy instance's local map.

The failing state sequence was:

1. A sandbox was public and the proxy cached `true`.
2. The owner changed the sandbox to private.
3. Cache invalidation failed, missed an instance, or could not address the active cache mode.
4. The proxy read the stale `true` value.
5. Anonymous traffic on an ordinary preview port bypassed authentication and reached the runner.

The system reported the sandbox as private while an older authorization decision remained active.

## 3. Reachability and impact

The path required a public-status entry to exist before the transition and remain stale afterward. It also required access to a non-reserved preview port. These are material prerequisites and make the finding conditional.

When they hold, an unauthenticated caller can continue reaching a service that the owner has made private. Sandbox previews can expose development servers, dashboards, application endpoints, source-derived output, or data available to the running service. The concrete loss depends on what the sandbox publishes on that port.

The stale decision lasts no longer than the surviving cache entry. The reviewed code used a one-hour time to live for public status. A shorter real window remains a privacy failure because the control plane has already told the owner that the sandbox is private.

## 4. How the KIT found it

The KIT modeled the change as an authorization state transition instead of reviewing cache code as a performance feature. Public documentation established that private previews require authorization. That intent produced a postcondition for `commitPublic(sandbox, false)`: later anonymous preview requests must not reach a route sink.

The claim placed the sandbox in public state with a cached `true`, committed the private state, and then evaluated an anonymous preview request. The candidate model changed the authoritative sandbox state while leaving the cache untouched. The next request still satisfied the proxy's cached-public branch.

That residual path identified the missing coupling between privacy state and cache state. Source review then confirmed three implementation facts needed for reachability:

- cached public status could authorize without an API check
- positive status survived for one hour
- invalidation failure did not block the transition

The K claim was constructed and was not machine-checked. Its purpose was to expose the unsafe state combination and direct source inspection to the operations that could create it.

## 5. Evidence status

The retained audit rechecked the complete source chain at the pinned revision. It confirmed the authorization branch, cache lifetime, best-effort invalidation, and local-cache option.

It did not run a complete Daytona deployment with a canary sandbox and a forced invalidation failure. The report therefore describes a source-confirmed conditional fail-open. It does not present the finding as a reproduced anonymous access event.

Deployment wiring determines which cache mode is active and how invalidation reaches proxy instances. The unsafe behavior remains present in the reviewed code whenever stale `true` survives the state change.

## 6. Repair direction and limits

A public-to-private transition should fail closed. The system can couple the state change to an authorization epoch, require proxy revalidation after the epoch changes, or deny anonymous access when invalidation cannot be confirmed. A short cache lifetime reduces exposure but does not give the transition a strict privacy guarantee.

Every proxy instance and cache mode needs the same invalidation semantics. Tests should hold a stale positive entry, change the sandbox to private, force invalidation failure, and verify that anonymous preview never reaches the runner.

This analysis covers the pinned revision. It makes no claim about current Daytona releases or a deployment that uses different cache and proxy wiring.
