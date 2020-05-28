# kubernetes 初探——日志收集


wp_id: 555
Status: publish
Date: 2018-09-30 04:58:00
Modified: 2020-05-16 11:24:43



在 K8S 中, 默认情况下, 日志分散在每个不同的 Pod, 可以使用 kubectl logs 命令查看, 但是当
Pod 被删除之后, 就无法查看该 Pod 的日志了, 所以需要一种永久性的保存方式.