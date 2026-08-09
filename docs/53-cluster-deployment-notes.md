# SyncTV 53 Cluster Deployment Notes

Deployment date: 2026-08-09

## Release

- Helm OCI chart: `oci://ghcr.io/synctv-org/synctv/charts/synctv`
- Chart and application version: `1.0.1`
- Namespace and release: `synctv/synctv`
- Ingress host: `synctv.192.168.12.53.nip.io`
- Ingress class: `nginx`
- Storage class: `openebs-hostpath`

The application, PostgreSQL, and Redis Pods were Ready after deployment. The public
server-info endpoint responded successfully through the HTTPS Ingress when the
client accepted the test certificate.

## Confirmed backend/deployment limitation

The release logs report `builtin_stun=degraded`: built-in STUN is enabled, while
`config.webrtc.stunExternalAddr` is empty. The Pod advertises a private cluster
address, so the server cannot start a client-reachable built-in STUN listener.
Configure a client-reachable public IP and port or DNS name in the deployment
values before relying on built-in STUN for WebRTC outside the cluster.

The frontend does not change this backend or deployment configuration.

## Confirmed backend realtime and permission limitations

- An administrator member override that explicitly adds
  `view_playback_history` is persisted in `adminAddedPermissions`, while the
  effective `self_room_member.permissions` snapshot still omits that bit. The
  other tested administrator overrides are reflected in the realtime snapshot.
- Room kicks and platform bans directly terminate the affected WebSocket
  connection. The resulting close carries the same observable information as
  a transport interruption, which makes immediate client-side classification
  unreliable. The realtime protocol should send a terminal event or an
  application-specific close code/reason before disconnecting the affected
  session.

These release `1.0.1` backend behaviors remain scoped to the backend.

## Startup observation

The application restarted twice while PostgreSQL completed its first startup. It
later reached Ready and remained stable; no persistent application failure was
observed.
