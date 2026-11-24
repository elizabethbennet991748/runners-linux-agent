# elizabethbennet991748 Runners Linux pool

Three independently scheduled standard Ubuntu 24.04 GitHub-hosted workers join
my existing Runners fleet. This is an independently owned repository, not a
fork. Its workflows check out only this repository. All three workers use
`github-linux-elizabeth` as their account pool tag and have distinct run labels.

## Schedules and replacement queues

| Worker | Workflow | Hourly trigger, UTC | Concurrency group |
| --- | --- | --- | --- |
| 1 | agent.yml | :10 | runners-single-agent |
| 2 | agent-2.yml | :40 | runners-single-agent-2 |
| 3 | agent-3.yml | :25 | runners-single-agent-3 |

Each worker has one active job and its own waiting replacement. GitHub scopes
these groups to this repository. `cancel-in-progress: false` preserves the
running job; a new pending request replaces an older pending request. Other
workers and account pools continue independently. There is no cross-account
GitHub API token or centralized runtime scheduler in these workflows.

Initial leases place renewals in gaps between current deadlines. Later
scheduled leases have a 345-minute budget within the 360-minute job timeout.
The native agent advertises its deadline using LIFETIME_MIN so the control
plane can plan departures before expiry. The wrapper also calls /api/drain
when stopping. Workload storage is ephemeral; durable data needs external storage.

## Configuration

`CONTROL_URL` is a repository variable pointing to the existing HTTPS fleet.
`JOIN_TOKEN` is an encrypted repository secret. The native agent is downloaded
from the control plane and verified against its SHA-256 digest. GitHub API and
job-session tokens are omitted from its environment, and checkout credentials
are not persisted.

Run either workflow manually from Actions to request that worker. The optional
runtime_minutes input accepts 30–345 minutes. Disable a worker's workflow to
stop replacement requests and cancel its active run to stop its current host.
The other workers and account pools are unaffected.

GitHub can delay or drop cron requests. Public-repository schedules can disable
after 60 days without repository activity. This is best-effort scheduling.

## Checks

`python3 -m unittest discover -s tests -v`

References: [limits](https://docs.github.com/en/actions/reference/limits),
[concurrency](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/control-workflow-concurrency),
[schedules](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule).
