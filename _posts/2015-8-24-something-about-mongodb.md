---

layout: post

title: 关于mongodb中collection的设计问题

---

最近一段时间mongodb使用比较频繁, 公司好多数据都从MySQL迁移到mongo. 因此, 便出现了一些数据库的设计问题.

比如有业务需求, 存储通讯录. 那么问题来了: 每个用户记录通讯录的方式都不相同, 如何将移动端的通讯录较为
