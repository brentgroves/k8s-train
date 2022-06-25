https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

CronJob
FEATURE STATE: Kubernetes v1.21 [stable]
A CronJob creates Jobs on a repeating schedule.

One CronJob object is like one line of a crontab (cron table) file. It runs a job periodically on a given schedule, written in Cron format.

Caution:
All CronJob schedule: times are based on the timezone of the kube-controller-manager.

If your control plane runs the kube-controller-manager in Pods or bare containers, the timezone set for the kube-controller-manager container determines the timezone that the cron job controller uses.