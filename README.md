# SuppressViewpunch

> [!WARNING]
> **This plugin is deprecated and no longer needed.**
>
> Following the Source SDK 2013 update
> ([ValveSoftware/source-sdk-2013@548e4e2](https://github.com/ValveSoftware/source-sdk-2013/commit/548e4e2b68d8f73f3655af8fb38a287d2bb198c8)),
> the landing viewpunch is now handled **client-side** and is no longer something
> the server needs to (or should) override.
>
> There is nothing left for this plugin to do. You can safely remove it from your
> server.

## What it used to do

It detoured `CGameMovement::PlayerRoughLandingEffects` on the server and
suppressed the camera "viewpunch" (screen kick) that happened when a player
slammed into the ground, gated behind the `sm_suppress_viewpunch` ConVar.

## Why it is no longer useful

The referenced engine update moved the punch-angle interpolation
(`m_Local.m_vecPunchAngle` / `m_Local.m_vecPunchAngleVel`) into the base
`C_BasePlayer` constructor. The rough-landing viewpunch effect is now applied and
interpolated on the client for every player type, so:

- The server-side detour no longer reliably controls what the player sees.
- Players who want to reduce or disable the effect can do so on their own client.

## Recommendation

Uninstall the plugin. If you still want server-enforced behaviour, it would need
to be re-implemented against the new client-side handling — the approach used here
is obsolete.
