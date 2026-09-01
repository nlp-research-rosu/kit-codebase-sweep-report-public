# Mastra default-auth custom route bypass

This is a confirmed P1. A custom API route documented as protected by default could reach its handler without authentication in direct server-adapter setups.

The reviewed revision is [`7a0d62f7ad60937ecc0f31993f1eab7281287fc5`](https://github.com/mastra-ai/mastra/commit/7a0d62f7ad60937ecc0f31993f1eab7281287fc5).

The evidence includes a pinned source trace, constructed K claims, and a targeted canary against Mastra's real authentication middleware.

## 1. System boundary

Mastra lets developers register custom API routes that call agents, models, workflows, and application tools. The route API [described authentication as the default](https://github.com/mastra-ai/mastra/blob/7a0d62f7ad60937ecc0f31993f1eab7281287fc5/packages/core/src/server/index.ts#L58-L117). A developer had to set `requiresAuth: false` to opt out.

Mastra also documented direct Express, Hono, Fastify, and Koa adapter construction. These adapters mounted routes from a Mastra instance and were expected to preserve the route security contract.

Every registered custom route had to authenticate before handler execution unless the route explicitly opted out.

## 2. Root cause

Authentication depended on metadata that some adapter paths did not construct.

Mastra used a custom-route authentication map to tell shared middleware which registered paths required authentication. Some framework adapters built that map from the route list. Direct adapter construction did not always derive it from `mastra.getServer().apiRoutes`.

The shared [`coreAuthMiddleware`](https://github.com/mastra-ai/mastra/blob/7a0d62f7ad60937ecc0f31993f1eab7281287fc5/packages/server/src/server/auth/helpers.ts#L347-L381) first asked whether the request path was protected. If neither the configured path patterns nor the custom-route map marked the path, it returned a pass result before authentication.

The [Express adapter](https://github.com/mastra-ai/mastra/blob/7a0d62f7ad60937ecc0f31993f1eab7281287fc5/server-adapters/express/src/index.ts#L618-L650) added another dependency on the same map. It invoked custom-route authentication only when the map identified the matched route as protected. A missing map therefore turned an omitted internal metadata step into a public route.

The failure affected custom paths outside the default protected pattern. A route such as `/completion` could retain its documented default `requiresAuth` value while bypassing the middleware that enforced it.

## 3. Reachability and impact

The vulnerable configuration used a documented direct adapter and a custom route without `requiresAuth: false`. No attacker account was required because the failure occurred before token authentication.

Impact depended on the handler. Mastra's public examples included completion routes that invoke model and agent execution. Applications can also place tools, workflow actions, storage access, or business operations behind custom handlers. An unauthenticated caller could therefore consume provider resources, retrieve application data, or trigger side effects that the developer expected Mastra to protect.

The P1 classification applies to default-protected routes that expose high-impact handlers. A health endpoint or another intentionally harmless handler would have lower practical impact even though the framework boundary failed in the same way.

## 4. How the KIT found it

The KIT wrote the route contract before tracing adapter code. Public documentation and type comments produced three obligations:

1. Custom routes are protected unless `requiresAuth` is explicitly false.
2. Direct adapters preserve the Mastra server configuration they mount.
3. Authentication and authorization occur before a protected handler.

The formal claim `AUTH-CUSTOM-DEFAULT` modeled the intended order. Authentication, authorization, and validation had to precede `handlerRan`. `AUTH-FAIL-CLOSED-DEFAULT` covered the missing-map case and required a default route to remain protected when no pattern or metadata entry resolved its status.

The implementation introduced an unstated precondition. The custom-route map had to exist even though direct-adapter documentation did not require callers to supply it. That precondition was absent from public intent and caused the fail-closed claim to break.

Next and TanStack initialization paths built the map from route definitions. The direct adapters did not, which made the missing step an implementation inconsistency rather than a necessary property of Mastra's route model.

The reduced K model was constructed and was not machine-checked. Its value here was to make the missing precondition explicit and connect it to handler execution.

## 5. Validation record

The retained audit called Mastra's actual `coreAuthMiddleware` with a default-auth `POST /completion` route and no custom-route map. The middleware advanced the request and made zero authentication calls.

The control added the expected map entry for the same route. The middleware then called authentication and returned a 401 response for the unauthenticated request.

This canary isolated the security decision without starting a public service or invoking a real model. It confirmed that the map controlled whether the default-auth route reached authentication. It did not rely on a reimplementation of the middleware.

## 6. Repair direction and limits

Every adapter should derive custom-route authentication metadata from the registered route list when callers omit it. A route with `requiresAuth !== false` should force authentication independently of broad path patterns. Explicit public routes must keep their opt-out behavior.

Adapter parity tests should cover default-protected and explicitly public custom routes across Express, Hono, Fastify, and Koa. A missing metadata map should fail closed.

This analysis covers the pinned revision and the tested middleware boundary. It does not characterize current Mastra releases or every deployment wrapper.
