# Reset offset on partitons
```bash
export BROKERS="kafka1:9091,kafka2:9091,kafka3:9091"

## Find lagged entries
kafka-consumer-groups --bootstrap-server $BROKERS --describe --all-groups

## Reset 
### To Earliest
kafka-consumer-groups --bootstrap-server $BROKERS --group {group} --topic {topic}:{partition} --reset-offsets --to-earliest --execute &&

### To Latest
kafka-consumer-groups --bootstrap-server $BROKERS --group {group} --topic {topic}:{partition} --reset-offsets --to-earliest --execute &&
```