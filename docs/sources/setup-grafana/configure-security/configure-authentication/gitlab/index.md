---
aliases:
  - ../../../auth/gitlab/
description: Grafana OAuthentication Guide
keywords:
  - grafana
  - configuration
  - documentation
  - oauth
labels:
  products:
    - cloud
    - enterprise
    - oss
menuTitle: GitLab OAuth2
title: Configure GitLab OAuth2 authentication
weight: 1000
---

# Configure GitLab OAuth2 authentication

{{< docs/shared lookup="auth/intro.md" source="grafana" version="<GRAFANA VERSION>" >}}

This topic describes how to configure GitLab OAuth2 authentication.

## Before you begin

To follow this guide:

- Ensure that you have access to the [Grafana configuration file]({{< relref "../../../configure-grafana#configuration-file-location" >}}).
- Ensure you know how to create a GitLab OAuth application. Consult GitLab's documentation on [creating a GitLab OAuth application](https://docs.gitlab.com/ee/integration/oauth_provider.html) for more information.

## Steps

To configure GitLab authentication with Grafana, follow these steps:

1. Create an OAuth application in GitLab.

   1. Set the redirect URI to `http://<my_grafana_server_name_or_ip>:<grafana_server_port>/login/gitlab`.

      Ensure that the Redirect URI is the complete HTTP address that you use to access Grafana via your browser, but with the appended path of `/login/gitlab`.

      For the Redirect URI to be correct, it might be necessary to set the `root_url` option in the `[server]`section of the Grafana configuration file. For example, if you are serving Grafana behind a proxy.

   1. Set the OAuth2 scopes to `openid`, `email` and `profile`.

1. Refer to the following table to update field values located in the `[auth.gitlab]` section of the Grafana configuration file:

   | Field                        | Description                                                                                  |
   | ---------------------------- | -------------------------------------------------------------------------------------------- |
   | `client_id`, `client_secret` | These values must match the client ID and client secret from your GitLab OAuth2 application. |
   | `enabled`                    | Enables GitLab authentication. Set this value to `true`.                                     |

   Review the list of other GitLab [configuration options]({{< relref "#configuration-options" >}}) and complete them, as necessary.

1. Optional: [Configure a refresh token]({{< relref "#configure-a-refresh-token" >}}):

   a. Enable `accessTokenExpirationCheck` feature toggle.

   b. Set `use_refresh_token` to `true` in `[auth.gitlab]` section in Grafana configuration file.

1. [Configure role mapping]({{< relref "#configure-role-mapping" >}}).
1. Optional: [Configure team synchronization]({{< relref "#configure-team-synchronization" >}}).
1. Restart Grafana.

   You should now see a GitLab login button on the login page and be able to log in or sign up with your GitLab accounts.

## Configuration options

The table below describes all GitLab OAuth configuration options. Like any other Grafana configuration, you can apply these options as environment variables.

| Setting                      | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Default                              |
| ---------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| `enabled`                    | Yes      | Whether GitLab OAuth authentication is allowed.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | `false`                              |
| `client_id`                  | Yes      | Client ID provided by your GitLab OAuth app.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |                                      |
| `client_secret`              | Yes      | Client secret provided by your GitLab OAuth app.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |                                      |
| `auth_url`                   | Yes      | Authorization endpoint of your GitLab OAuth provider. If you use your own instance of GitLab instead of gitlab.com, adjust `auth_url` by replacing the `gitlab.com` hostname with your own.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | `https://gitlab.com/oauth/authorize` |
| `token_url`                  | Yes      | Endpoint used to obtain GitLab OAuth access token. If you use your own instance of GitLab instead of gitlab.com, adjust `token_url` by replacing the `gitlab.com` hostname with your own.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | `https://gitlab.com/oauth/token`     |
| `api_url`                    | No       | Grafana uses `<api_url>/user` endpoint to obtain GitLab user information compatible with [OpenID UserInfo](https://connect2id.com/products/server/docs/api/userinfo).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | `https://gitlab.com/api/v4`          |
| `name`                       | No       | Name used to refer to the GitLab authentication in the Grafana user interface.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | `GitLab`                             |
| `icon`                       | No       | Icon used for GitLab authentication in the Grafana user interface.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | `gitlab`                             |
| `scopes`                     | No       | List of comma or space-separated GitLab OAuth scopes.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | `openid email profile`               |
| `allow_sign_up`              | No       | Whether to allow new Grafana user creation through GitLab login. If set to `false`, then only existing Grafana users can log in with GitLab OAuth.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | `true`                               |
| `auto_login`                 | No       | Set to `true` to enable users to bypass the login screen and automatically log in. This setting is ignored if you configure multiple auth providers to use auto-login.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | `false`                              |
| `role_attribute_path`        | No       | [JMESPath](http://jmespath.org/examples.html) expression to use for Grafana role lookup. Grafana will first evaluate the expression using the GitLab OAuth token. If no role is found, Grafana creates a JSON data with `groups` key that maps to groups obtained from GitLab's `/oauth/userinfo` endpoint, and evaluates the expression using this data. Finally, if a valid role is still not found, the expression is evaluated against the user information retrieved from `api_url/users` endpoint and groups retrieved from `api_url/groups` endpoint. The result of the evaluation should be a valid Grafana role (`Viewer`, `Editor`, `Admin` or `GrafanaAdmin`). For more information on user role mapping, refer to [Configure role mapping]({{< relref "#configure-role-mapping" >}}). |                                      |
| `role_attribute_strict`      | No       | Set to `true` to deny user login if the Grafana role cannot be extracted using `role_attribute_path`. For more information on user role mapping, refer to [Configure role mapping]({{< relref "#configure-role-mapping" >}}).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | `false`                              |
| `allow_assign_grafana_admin` | No       | Set to `true` to enable automatic sync of the Grafana server administrator role. If this option is set to `true` and the result of evaluating `role_attribute_path` for a user is `GrafanaAdmin`, Grafana grants the user the server administrator privileges and organization administrator role. If this option is set to `false` and the result of evaluating `role_attribute_path` for a user is `GrafanaAdmin`, Grafana grants the user only organization administrator role. For more information on user role mapping, refer to [Configure role mapping]({{< relref "#configure-role-mapping" >}}).                                                                                                                                                                                        | `false`                              |
| `skip_org_role_sync`         | No       | Set to `true` to stop automatically syncing user roles.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | `false`                              |
| `allowed_domains`            | No       | List of comma or space-separated domains. User must belong to at least one domain to log in.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |                                      |
| `allowed_groups`             | No       | List of comma or space-separated groups. The user should be a member of at least one group to log in.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |                                      |
| `tls_skip_verify_insecure`   | No       | If set to `true`, the client accepts any certificate presented by the server and any host name in that certificate. _You should only use this for testing_, because this mode leaves SSL/TLS susceptible to man-in-the-middle attacks.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | `false`                              |
| `tls_client_cert`            | No       | The path to the certificate.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |                                      |
| `tls_client_key`             | No       | The path to the key.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |                                      |
| `tls_client_ca`              | No       | The path to the trusted certificate authority list.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |                                      |
| `use_pkce`                   | No       | Set to `true` to use [Proof Key for Code Exchange (PKCE)](https://datatracker.ietf.org/doc/html/rfc7636). Grafana uses the SHA256 based `S256` challenge method and a 128 bytes (base64url encoded) code verifier.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | `true`                               |
| `use_refresh_token`          | No       | Set to `true` to use refresh token and check access token expiration. The `accessTokenExpirationCheck` feature toggle should also be enabled to use refresh token.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | `true`                               |

### Configure a refresh token

> Available in Grafana v9.3 and later versions.

> **Note:** This feature is behind the `accessTokenExpirationCheck` feature toggle.

When a user logs in using an OAuth provider, Grafana verifies that the access token has not expired. When an access token expires, Grafana uses the provided refresh token (if any exists) to obtain a new access token.

Grafana uses a refresh token to obtain a new access token without requiring the user to log in again. If a refresh token doesn't exist, Grafana logs the user out of the system after the access token has expired.

By default, GitLab provides a refresh token.

Refresh token fetching and access token expiration check is enabled by default for the GitLab provider since Grafana v10.1.0 if the `accessTokenExpirationCheck` feature toggle is enabled. If you would like to disable access token expiration check then set the `use_refresh_token` configuration value to `false`.

> **Note:** The `accessTokenExpirationCheck` feature toggle will be removed in Grafana v10.3.0 and the `use_refresh_token` configuration value will be used instead for configuring refresh token fetching and access token expiration check.

### Configure allowed groups

To limit access to authenticated users that are members of one or more [GitLab
groups](https://docs.gitlab.com/ce/user/group/index.html), set `allowed_groups`
to a comma or space-separated list of groups.

GitLab's groups are referenced by the group name. For example, `developers`. To reference a subgroup `frontend`, use `developers/frontend`.
Note that in GitLab, the group or subgroup name does not always match its display name, especially if the display name contains spaces or special characters.
Make sure you always use the group or subgroup name as it appears in the URL of the group or subgroup.

## Configure role mapping

Unless `skip_org_role_sync` option is enabled, the user's role will be set to the role retrieved from GitLab upon user login.

The user's role is retrieved using a [JMESPath](http://jmespath.org/examples.html) expression from the `role_attribute_path` configuration option.
To map the server administrator role, use the `allow_assign_grafana_admin` configuration option.
Refer to [configuration options]({{< relref "#configuration-options" >}}) for more information.

If no valid role is found, the user is assigned the role specified by [the `auto_assign_org_role` option]({{< relref "../../../configure-grafana#auto_assign_org_role" >}}).
You can disable this default role assignment by setting `role_attribute_strict = true`.
This setting denies user access if no role or an invalid role is returned.

To ease configuration of a proper JMESPath expression, go to [JMESPath](http://jmespath.org/) to test and evaluate expressions with custom payloads.

### Role mapping examples

This section includes examples of JMESPath expressions used for role mapping.

#### Map roles using user information from OAuth token

In this example, the user with email `admin@company.com` has been granted the `Admin` role.
All other users are granted the `Viewer` role.

```ini
role_attribute_path = email=='admin@company.com' && 'Admin' || 'Viewer'
```

#### Map roles using groups

In this example, the user from GitLab group 'example-group' have been granted the `Editor` role.
All other users are granted the `Viewer` role.

```ini
role_attribute_path = contains(groups[*], 'example-group') && 'Editor' || 'Viewer'
```

#### Map server administrator role

In this example, the user with email `admin@company.com` has been granted the `Admin` organization role as well as the Grafana server admin role.
All other users are granted the `Viewer` role.

```bash
role_attribute_path = email=='admin@company.com' && 'GrafanaAdmin' || 'Viewer'
```

#### Map one role to all users

In this example, all users will be assigned `Viewer` role regardless of the user information received from the identity provider.

```ini
role_attribute_path = "'Viewer'"
skip_org_role_sync = false
```

## Configure team synchronization

> **Note:** Available in [Grafana Enterprise]({{< relref "../../../../introduction/grafana-enterprise" >}}) and [Grafana Cloud](/docs/grafana-cloud/).

By using Team Sync, you can map GitLab groups to teams within Grafana. This will automatically assign users to the appropriate teams.
Teams for each user are synchronized when the user logs in.

GitLab groups are referenced by the group name. For example, `developers`. To reference a subgroup `frontend`, use `developers/frontend`.
Note that in GitLab, the group or subgroup name does not always match its display name, especially if the display name contains spaces or special characters.
Make sure you always use the group or subgroup name as it appears in the URL of the group or subgroup.

To learn more about Team Sync, refer to [Configure team sync]({{< relref "../../configure-team-sync" >}}).

## Example of GitLab configuration in Grafana

This section includes an example of GitLab configuration in the Grafana configuration file.

```bash
[auth.gitlab]
enabled = true
allow_sign_up = true
auto_login = false
client_id = YOUR_GITLAB_APPLICATION_ID
client_secret = YOUR_GITLAB_APPLICATION_SECRET
scopes = openid email profile
auth_url = https://gitlab.com/oauth/authorize
token_url = https://gitlab.com/oauth/token
api_url = https://gitlab.com/api/v4
role_attribute_path = contains(groups[*], 'example-group') && 'Editor' || 'Viewer'
role_attribute_strict = false
allow_assign_grafana_admin = false
allowed_groups = ["admins", "software engineers", "developers/frontend"]
allowed_domains = mycompany.com mycompany.org
tls_skip_verify_insecure = false
use_pkce = true
use_refresh_token = true
```
