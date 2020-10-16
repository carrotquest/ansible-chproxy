# Ansible ClickHouse proxy

Install chproxy binary as systemd service.
More info at chproxy project: [Vertamedia/chproxy](https://github.com/Vertamedia/chproxy)

## Role Variables

`chproxy_node` - enable/disable chproxy.
`chproxy_bind_address` - listen on address.
`chproxy_bind_port` - listen on port.

Default variables for a config generation:

```yaml
chproxy_caches:
  - name: "shortterm"
    dir: "{{ chproxy.data_dir }}/shortterm"
    max_size: 1Gb
    expire: 160s

chproxy_network_groups:
  - name: "local-net"
    networks: ["127.0.0.0/8"]
  - name: "base-net"
    networks: ["10.0.0.0/24"]
  - name: "monitoring-host"
    networks: ["10.0.0.10"]

chproxy_server:
  http:
    listen_addr: "{{ chproxy_bind_address }}:{{ chproxy_bind_port }}"
    allowed_networks: ["base-net", "local-net"]
    read_timeout: 5m
    write_timeout: 10m
    idle_timeout: 15m
  metrics:
    allowed_networks: ["monitoring-host", "local-net"]

chproxy_users:
  - name: "chproxy_first_username"
    password: "chproxy_first_password"
    to_cluster: "clickhouse_first_cluster"
    to_user: "chproxy_first_username"
    cache: "shortterm"
    requests_per_minute: 10
    max_queue_size: 10
    max_queue_time: 10s
    max_concurrent_queries: 2
    max_execution_time: 1m
  - name: "chproxy_second_username"
    password: "chproxy_second_password"
    to_cluster: "clickhouse_first_cluster"
    to_user: "chproxy_second_username"
    cache: "shortterm"
    requests_per_minute: 10
    max_queue_size: 10
    max_queue_time: 10s
    max_concurrent_queries: 2
    max_execution_time: 1m

chproxy_clusters:
  - name: "clickhouse_first_cluster"
    scheme: "http"
    nodes:
      - "10.0.0.1:8123"
      - "10.0.0.2:8123"
    heartbeat_interval: 5s
    kill_query_user:
      name: "clickhouse_default_username"
      password: "clickhouse_default_password"
    # Configuration for ClickHouse cluster users.
    users:
      - name: "clickhouse_first_username"
        password: "clickhouse_first_password"
        max_concurrent_queries: 100
        max_execution_time: 15m
      - name: "clickhouse_second_username"
        password: "clickhouse_second_password"
        max_concurrent_queries: 100
        max_execution_time: 15m
```

## Example Playbook

```yml
- hosts: localhost
  remote_user: root
  become: yes
  roles:
    - ansible-chproxy
```

## License

MIT License

## Author Information

Carrol Cox <carrol@protonmail.com>
