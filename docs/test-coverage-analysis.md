# Test Coverage Analysis

## Executive Summary

The `rprakashg.vpac` Ansible collection has **16 roles** and **16 test files**, but the tests are effectively non-functional. No role is meaningfully exercised by CI, and the test job in the CI pipeline does not execute any playbooks at all. The sections below detail each problem and propose concrete improvements.

---

## Critical Issues

### 1. All test.yml files reference non-existent role paths

Every `tests/test.yml` uses a relative path to a role name that does not exist in the repository. These tests would fail instantly if run.

| Role directory | Referenced role in test.yml | Correct FQCN |
|---|---|---|
| `configure_bridge_interfaces` | `./roles/configure_networking` | `rprakashg.vpac.configure_bridge_interfaces` |
| `configure_centralized_management_system` | `./roles/configure_aap` | `rprakashg.vpac.configure_centralized_management_system` |
| `create_image_installer` | `./roles/build_iso` | `rprakashg.vpac.create_image_installer` |
| `deploy_linux_vm` | `./roles/deploy_vm` | `rprakashg.vpac.deploy_linux_vm` |
| `deploy_ssc600_vm` | `./roles/deploy_ssc600sw` | `rprakashg.vpac.deploy_ssc600_vm` |
| `deploy_windows_vm` | `./roles/create_windows_vm` | `rprakashg.vpac.deploy_windows_vm` |
| `inject_ks_into_iso` | `./roles/customize_iso` | `rprakashg.vpac.inject_ks_into_iso` |
| `prepare_system_for_ssc600_vm` | `./roles/prepare_system_for_ssc600sw` | `rprakashg.vpac.prepare_system_for_ssc600_vm` |
| `setup_centralized_management_system` | `./roles/install_aap` | `rprakashg.vpac.setup_centralized_management_system` |

**Fix:** Update every `tests/test.yml` to use the correct FQCN for the role being tested.

---

### 2. The CI `test` job never runs any tests

In `.github/workflows/ci.yaml`, the `test` job installs `ansible-core` and `amazon.aws` but then immediately cleans up the subscription without executing a single playbook or test file. This means CI provides zero functional test coverage.

```yaml
# Current (incomplete) test job — no test execution step
- name: Install ansible core
  run: dnf install -y ansible-core python3
- name: Install required ansible collections
  run: ansible-galaxy collection install amazon.aws
- name: Clean up the subscription   # jumps straight to cleanup
  run: subscription-manager unregister
```

**Fix:** Add a step that actually runs the test playbooks, for example using Molecule or a direct `ansible-playbook` call with `--check` mode and test-specific inventory/variable files.

---

### 3. Lint failures are non-blocking

Both `ansible-lint` and `yamllint` run with `continue-on-error: true`. This means quality failures are silently ignored and never block a merge.

**Fix:** Remove `continue-on-error: true` from both linting steps once the codebase passes lint cleanly, so regressions are caught.

---

## Structural Test Gaps

### 4. No assertions in any test playbook

None of the 16 `test.yml` files contain `ansible.builtin.assert` tasks. A test that applies a role but verifies nothing cannot confirm that the role did the right thing.

**Recommended assertions to add per role:**

| Role | Example assertions |
|---|---|
| `configure_bridge_interfaces` | NIC exists, bridge connection is active, IP address is reachable |
| `create_cloudinit_iso` | ISO file exists at expected path, is a valid ISO9660 image |
| `deploy_linux_vm` | libvirt domain is defined, VM is in `running` state |
| `register_system` | `/etc/insights-client/.registered` file exists, `subscription-manager status` returns `Current` |
| `prepare_system_for_rt` | RT kernel package is installed, tuned profile is active, hugepages are configured |
| `setup_centralized_management_system` | Gitea container is running, InfluxDB HTTP endpoint responds, Loki service is active |
| `deploy_otelcollector` | OTel Collector service is running, config file exists |
| `setup_imagebuilder` | `osbuild-composer` service is enabled and running |

---

### 5. No variable injection in test playbooks

Roles like `register_system` require sensitive variables (`rh_user`, `rh_user_password`) and infrastructure-specific variables (`processbus_nic`, IP addresses, etc.) that are never set in the test playbooks. Running these tests as-is would cause immediate failures.

**Fix:** Add `vars:` blocks in each test playbook with safe default values for local/CI testing (using `check_mode` or mocked secrets), and store sensitive values in GitHub Actions secrets mapped to Ansible variables.

---

### 6. No idempotency testing

Ansible roles must be idempotent — running them twice should produce the same result with no changes on the second run. No test verifies this.

**Fix:** In the CI pipeline, run each test playbook twice and assert that the second run reports `changed=0`.

---

### 7. No Molecule testing framework

The project has no [Molecule](https://ansible.readthedocs.io/projects/molecule/) configuration. Molecule provides:
- Isolated container/VM environments for each role
- A standardised converge → verify → idempotency lifecycle
- Easy integration with GitHub Actions

**Fix:** Add a `molecule/` directory per role (or a shared scenario at collection level) using the `docker` or `podman` driver. This would allow roles to be tested without requiring actual RHEL subscriptions in CI.

---

### 8. `configure_ha` role is a stub

`roles/configure_ha/tasks/main.yml` contains no tasks. The test for it would do nothing even if run correctly.

**Fix:** Either implement the role's tasks, or add a clear `# TODO` comment explaining what HA configuration is planned and skip it in CI until it is implemented.

---

### 9. Jinja2 templates are not tested

18 Jinja2 templates (libvirt XML, cloud-init, kickstart, systemd units, container configs, etc.) are not independently tested. Template rendering errors would only surface at runtime.

**Fix:** Add test tasks that render each template with representative variable values and validate the output — for example:
- Validate libvirt domain XML with `xmllint`
- Validate cloud-init files with `cloud-init devel schema`
- Validate kickstart files with `ksvalidator` (from `pykickstart`)

---

### 10. Missing integration/playbook-level tests

The 7 top-level playbooks in `playbooks/` have no tests at all. These are the user-facing entry points for the collection and represent the most realistic test scenarios.

**Fix:** Add playbook-level integration tests that exercise realistic combinations of roles:
- `create_iso.yml` → `inject_ks_into_iso` + `create_cloudinit_iso`
- `deploy_ssc600sw.yml` → `prepare_system_for_ssc600_vm` + `deploy_ssc600_vm`
- `prepare_system_for_rt.yml` → `register_system` + `prepare_system_for_rt`

---

## Prioritised Recommendations

| Priority | Action | Effort |
|---|---|---|
| **P0** | Fix wrong role names in all `tests/test.yml` files | Low |
| **P0** | Add actual test execution step to the CI `test` job | Low |
| **P1** | Add `assert` tasks to each test playbook | Medium |
| **P1** | Inject required variables into test playbooks | Medium |
| **P1** | Remove `continue-on-error: true` from linting | Low |
| **P2** | Add Molecule to enable isolated, repeatable testing | High |
| **P2** | Add idempotency checks (run each playbook twice) | Low |
| **P2** | Add template validation tasks | Medium |
| **P3** | Implement `configure_ha` role tasks | High |
| **P3** | Add integration tests for top-level playbooks | High |
