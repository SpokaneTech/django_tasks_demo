---
title: "DEP 0014: Django Background Workers"
marp: true
html: true
theme: gaia

---
<style>
section {
    --color-background: #ff00ff22 !important;
    --color-foreground: white;
}
section code {
  --color-background: #bbb; /* text color, weirdly */
  --color-dimmed: none;
}
section table {
  font-size: 0.65rem;

  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
section table thead th {
  --color-background: black;
}
section table tbody td {
  vertical-align: baseline;
}
.emphasis {
    font-weight: bolder;
    color: #11779C;
}
h1 {
    font-size: 1.8rem;
    font-weight: 600;
    margin-bottom: 1rem;
}
h2 {
    font-size: 2rem;
}
ul {
    margin-top: 0px;
    margin-left: 8px;
}
</style>

<!-- _class: lead -->
<h1>DEP 0014:<br>Django Background Workers</h1>

Joe Riddle

---

# Outline

- [Use cases](#use-cases)
- [The state of background workers](#the-state-of-background-workers)
- [DEP 0014](#dep-0014)
- [Task Backends](#task-backends)
- [Demo 👨‍💻](#demo-)
- [Features](#features)
- [Questions 🎤](#questions-)

--- 

# Use cases

* Sending emails
* Web scraping
* Nightly jobs
* Anything that takes a **long time**
* Anything that needs to **run periodically**

--- 

# The state of background workers

- Celery
- RQ, Dramatiq, Huey, ...
- Cloud native solutions
  - [AWS Lambda](https://aws.amazon.com/lambda/) + [EventBridge Scheduler](https://docs.aws.amazon.com/scheduler/latest/UserGuide/what-is-scheduler.html)
  - [Azure Functions](https://learn.microsoft.com/en-us/azure/azure-functions/)

---

# DEP 0014

> This proposal sets out to provide an interface and base implementation for long-running background tasks in Django.

— https://github.com/django/deps/blob/main/accepted/0014-background-workers.rst

--- 

# DEP 0014 — History
* [RFC 72: Background workers](https://github.com/wagtail/rfcs/pull/72)
  * Created by Jake Howard at [Wagtail CMS](https://wagtail.org/) in **2021**
  * [Added](https://github.com/wagtail/rfcs/pull/72#issuecomment-1926955690) to Wagtail roadmap in 2024
* 2 days later, [Carlton Gibson](https://noumenal.es/) drops a [💣](https://fosstodon.org/@carlton/111888693854550861)
  * "Hi @wagtail do you think we can make this RFC a proposal for Django itself?"
* _3 days later_, Jake Howard creates the [DEP 0014 PR](https://github.com/django/deps/pull/86)
  * ...which is accepted and merged ~3 months later

---

# DEP 0014 — Benefits
- lower barrier to entry for offloading computation to the background
- allow third-party libraries to offload some of their computation

---

# DEP 0014 — Task Backends

Django will ship with three task backends:
1) ImmediateBackend — Run tasks immediately without a background process. **Useful for transitioning.**
2) DummyBackend — Do nothing but store tasks in memory. **Useful for tests.**
3) DatabaseBackend — Use the ORM as a task store. **Useful for _realsies_**.

--- 

<!-- _class: lead -->
# Demo 👨‍💻

<!-- http://127.0.0.1:8000/admin/django_tasks_database/dbtaskresult/ -->

---

| Features             | DEP 0014                                                                                            | Celery                                                                          | django-rq (Redis)                                                                                                            | Huey                                                                              |
| -------------------- | --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Background tasks     | ✅                                                                                                   | ✅                                                                               | ✅                                                                                                                            | ✅                                                                                 |
| Deferred tasks       | ✅                                                                                                   | ✅                                                                               | ✅                                                                                                                            | ✅                                                                                 |
| Recurring scheduling | ❌                                                                                                   | ✅ With [django-celery-beat](https://github.com/celery/django-celery-beat)       | ✅ With [rq-scheduler](https://github.com/rq/rq-scheduler)                                                                    | ✅                                                                                 |
| Task results         | ✅                                                                                                   | ✅ With [django-celery-results](https://pypi.org/project/django-celery-results/) | ✅                                                                                                                            | ✅                                                                                 |
| Automatic retry      | ❌                                                                                                   | ✅                                                                               | ✅                                                                                                                            | ✅                                                                                 |
| Timeouts             | ❌                                                                                                   | ✅                                                                               | ✅                                                                                                                            | ✅                                                                                 |
| Hooks                | ❌<br>but [Django Signals](https://github.com/RealOrangeOne/django-tasks?tab=readme-ov-file#signals) | ✅<br>[Signals](https://docs.celeryq.dev/en/stable/userguide/signals.html)       | ✅<br>[dependencies](https://python-rq.org/docs/#job-dependencies) and [callbacks](https://python-rq.org/docs/#job-callbacks) | ✅<br>[pipelines](https://huey.readthedocs.io/en/latest/guide.html#task-pipelines) |
| Monitoring           | ❌                                                                                                   | ✅ various ways, [Flower](https://flower.readthedocs.io/en/latest/)              | ✅ With [rq-dashboard](https://github.com/Parallels/rq-dashboard)                                                             | ❌                                                                                 |
| Setup effort         | Less:<br>management command (worker)                                                                | Lots:<br>queue, worker, beat                                                    | Some:<br>redis, worker                                                                                                       | Some:<br>redis, worker                                                            |
| Brokers              | Database                                                                                            | RabbitMQ, Redis, others                                                         | Redis                                                                                                                        | Redis, sqlite, FS, in-memory                                                      |

---

<!-- _class: lead -->
# Questions 🎤
