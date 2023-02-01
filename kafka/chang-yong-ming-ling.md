# 常用命令

## 消费者

+ 查看消费组情况

```
./kafka-consumer-groups.sh --bootstrap-server wlts-2-kvm-32c128g-6:29092,wlts-2-kvm-32c128g-2:29092,wlts-2-kvm-32c128g-3:29092,wlts-2-kvm-32c128g-4:29092,wlts-2-kvm-32c128g-5:29092 --describe --group test-consumer-group
```

- 消费数据
  
  ```
  ./kafka-console-consumer.sh --bootstrap-server wlts-2-kvm-32c128g-6:29092,wlts-2-kvm-32c128g-2:29092,wlts-2-kvm-32c128g-3:29092,wlts-2-kvm-32c128g-4:29092,wlts-2-kvm-32c128g-5:29092 --topic branch-nx-orglog --from-beginning --consumer-property group.id=test-consumer-group?? |grep 10423932156019981
  
  ./kafka-console-consumer.sh --bootstrap-server wlts-2-kvm-32c128g-6:29092,wlts-2-kvm-32c128g-2:29092,wlts-2-kvm-32c128g-3:29092,wlts-2-kvm-32c128g-4:29092,wlts-2-kvm-32c128g-5:29092 --topic branch-nx-orglog  --offset 1890343 --consumer-property group.id=test-consumer-group |grep 10.229.100.2 >10.229.100.2.txt
  
  ./kafka-console-consumer.sh --bootstrap-server wlts-2-kvm-32c128g-6:29092,wlts-2-kvm-32c128g-2:29092,wlts-2-kvm-32c128g-3:29092,wlts-2-kvm-32c128g-4:29092,wlts-2-kvm-32c128g-5:29092 --topic branch-nx-orglog  --property print.key=true --consumer-property group.id=test
  ```