# Tracing

wp_id: 588
Status: draft
Date: 2018-06-22 04:37:00
Modified: 2020-05-16 11:09:09

Yifei's Notes

线程的每次执行，都视为一个 trace, 因此在每次执行开始时，都 start_span 以创建一个 root span（也相应创建了一个新的 trace). 每个 span 内部执行的多个操作，也建模为多个子 span（使用 start_child_span 创建）

https://wu-sheng.gitbooks.io/opentracing-io/content/
https://gianarb.it/blog/faq-distributed-tracing