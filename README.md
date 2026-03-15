# prometheus immich-stats-exporter

Export immich statistics to prometheus with [json_exporter](https://github.com/prometheus-community/json_exporter). No custom code required, just configuration.

## Metrics

The exporter collects the following metrics:

| Metric Name | Type | Labels | Description |
|---|---|---|---|
| `immich_server_photos` | gauge | | Total number of photos on the server |
| `immich_server_videos` | gauge | | Total number of videos on the server |
| `immich_server_usage_bytes` | gauge | | Total storage usage in bytes |
| `immich_server_usage_photos_bytes` | gauge | | Storage used by photos in bytes |
| `immich_server_usage_videos_bytes` | gauge | | Storage used by videos in bytes |
| `immich_user_photos` | gauge | `user_id`, `user_name` | Number of photos per user |
| `immich_user_videos` | gauge | `user_id`, `user_name` | Number of videos per user |
| `immich_user_usage_bytes` | gauge | `user_id`, `user_name` | Storage usage per user in bytes |
| `immich_user_usage_photos_bytes` | gauge | `user_id`, `user_name` | Storage used by photos per user in bytes |
| `immich_user_usage_videos_bytes` | gauge | `user_id`, `user_name` | Storage used by videos per user in bytes |

### Usage

1. Create an [API key in Immich](https://my.immich.app/user-settings?isOpen=api-keys). Only the `server.statistics` permission is required.

1. Add the module to your json_exporter's `config.yml`:
    ```yml
    modules:
      immich_statistics:
        headers:
          Accept: application/json
          x-api-key: "YOUR_IMMICH_API_KEY"
        metrics:
          - name: immich_server
            type: object
            help: Immich server-wide media statistics
            path: "{$}"
            valuetype: gauge
            values:
              photos: "{.photos}"
              videos: "{.videos}"
              usage_bytes: "{.usage}"
              usage_photos_bytes: "{.usagePhotos}"
              usage_videos_bytes: "{.usageVideos}"

          - name: immich_user
            type: object
            help: Immich per-user media statistics
            path: "{.usageByUser[*]}"
            valuetype: gauge
            labels:
              user_id: "{.userId}"
              user_name: "{.userName}"
            values:
              photos: "{.photos}"
              videos: "{.videos}"
              usage_bytes: "{.usage}"
              usage_photos_bytes: "{.usagePhotos}"
              usage_videos_bytes: "{.usageVideos}"
    ```

    <details>
    <summary>Start json_exporter with podman or docker</summary>

    ```bash
    docker run -d \
      --name json-exporter \
      -p 7979:7979 \
      -v /path/to/config.yml:/config.yml:ro \
      quay.io/prometheuscommunity/json-exporter:latest \
      --config.file=/config.yml
    ```

    Compose:
    ```yaml
    services:
      json-exporter:
        image: quay.io/prometheuscommunity/json-exporter:latest
        ports:
          - "7979:7979"
        volumes:
          - /path/to/config.yml:/config.yml:ro
        command:
          - --config.file=/config.yml
    ```

    Podman Quadlet
    ```bash
    [Unit]
    Description=Prometheus JSON Exporter
    After=network-online.target
    Wants=network-online.target

    [Container]
    Image=quay.io/prometheuscommunity/json-exporter:latest
    PublishPort=7979:7979
    Network=internal.network
    Volume=/path/to/config.yml:/config.yml:ro

    Exec=--config.file=/config.yml

    # hardening
    ReadOnly=true
    NoNewPrivileges=true
    DropCapability=all
    PidsLimit=128

    [Service]
    Restart=on-failure
    ```

    </details>



1. Visit http://localhost:7979/probe?module=immich_statistics?target=http://your_immich_url:2283/api/server/statistics to verify that the metrics are being exported correctly. You should see output similar to this:

    <details>
    <summary>Metrics</summary>

    ```metrics
    # HELP immich_server_photos Immich server-wide media statistics
    # TYPE immich_server_photos gauge
    immich_server_photos 89494
    # HELP immich_server_usage_bytes Immich server-wide media statistics
    # TYPE immich_server_usage_bytes gauge
    immich_server_usage_bytes 1.9e+12
    # HELP immich_server_usage_photos_bytes Immich server-wide media statistics
    # TYPE immich_server_usage_photos_bytes gauge
    immich_server_usage_photos_bytes 2.6610944715e+11
    # HELP immich_server_usage_videos_bytes Immich server-wide media statistics
    # TYPE immich_server_usage_videos_bytes gauge
    immich_server_usage_videos_bytes 2.25718340831e+11
    # HELP immich_server_videos Immich server-wide media statistics
    # TYPE immich_server_videos gauge
    immich_server_videos 1094
    # HELP immich_user_photos Immich per-user media statistics
    # TYPE immich_user_photos gauge
    immich_user_photos{user_id="046728bc-3122-4208-a7d4-43426258280e",user_name="user1"} 22781
    immich_user_photos{user_id="0e5f182b-423a-4f12-bf13-5a9d3c55cdeb",user_name="user2"} 0
    immich_user_photos{user_id="1bc1c8a6-94d2-466b-9509-12174b3eec1c",user_name="user3"} 45935
    immich_user_photos{user_id="28b6572d-1aca-4fab-80aa-069ef03ac24e",user_name="user4"} 20778
    # HELP immich_user_usage_bytes Immich per-user media statistics
    # TYPE immich_user_usage_bytes gauge
    immich_user_usage_bytes{user_id="046728bc-3122-4208-a7d4-43426258280e",user_name="user1"} 9.7962460034e+10
    immich_user_usage_bytes{user_id="0e5f182b-423a-4f12-bf13-5a9d3c55cdeb",user_name="user2"} 0
    immich_user_usage_bytes{user_id="1bc1c8a6-94d2-466b-9509-12174b3eec1c",user_name="user3"} 1.09684467906e+11
    immich_user_usage_bytes{user_id="28b6572d-1aca-4fab-80aa-069ef03ac24e",user_name="user4"} 1.15631302037e+11
    # HELP immich_user_usage_photos_bytes Immich per-user media statistics
    # TYPE immich_user_usage_photos_bytes gauge
    immich_user_usage_photos_bytes{user_id="046728bc-3122-4208-a7d4-43426258280e",user_name="user1"} 5.0195785047e+10
    immich_user_usage_photos_bytes{user_id="0e5f182b-423a-4f12-bf13-5a9d3c55cdeb",user_name="user2"} 0
    immich_user_usage_photos_bytes{user_id="1bc1c8a6-94d2-466b-9509-12174b3eec1c",user_name="user3"} 8.3719727811e+10
    immich_user_usage_photos_bytes{user_id="28b6572d-1aca-4fab-80aa-069ef03ac24e",user_name="user4"} 7.5685561382e+10
    # HELP immich_user_usage_videos_bytes Immich per-user media statistics
    # TYPE immich_user_usage_videos_bytes gauge
    immich_user_usage_videos_bytes{user_id="046728bc-3122-4208-a7d4-43426258280e",user_name="user1"} 4.7766674987e+10
    immich_user_usage_videos_bytes{user_id="0e5f182b-423a-4f12-bf13-5a9d3c55cdeb",user_name="user2"} 0
    immich_user_usage_videos_bytes{user_id="1bc1c8a6-94d2-466b-9509-12174b3eec1c",user_name="user3"} 2.5964740095e+10
    immich_user_usage_videos_bytes{user_id="28b6572d-1aca-4fab-80aa-069ef03ac24e",user_name="user4"} 3.9945740655e+10
    # HELP immich_user_videos Immich per-user media statistics
    # TYPE immich_user_videos gauge
    immich_user_videos{user_id="046728bc-3122-4208-a7d4-43426258280e",user_name="user1"} 184
    immich_user_videos{user_id="0e5f182b-423a-4f12-bf13-5a9d3c55cdeb",user_name="user2"} 0
    immich_user_videos{user_id="1bc1c8a6-94d2-466b-9509-12174b3eec1c",user_name="user3"} 477
    immich_user_videos{user_id="28b6572d-1aca-4fab-80aa-069ef03ac24e",user_name="user4"} 433
    ```

    </details>



2. Add the json_exporter endpoint to your Prometheus configuration:
    prometheus.yml:
    ```yaml
    scrape_configs:
      - job_name: immich
        scrape_interval: 60s
        metrics_path: /probe
        params:
          module: [immich_statistics]
        static_configs:
          - targets:
              - http://your_immich_url:2283/api/server/statistics
        relabel_configs:
          - source_labels: [__address__]
            target_label: __param_target
          - source_labels: [__param_target]
            target_label: instance
          - target_label: __address__
            replacement: host.containers.internal:7979
    ```

    Note `your_immich_url` should be the URL where the json_exporter can reach the Immich API. If you are running json_exporter and immich in a container, use the internal container hostname of the Immich container (e.g. `immich:2283`).


1. Restart Prometheus to apply the changes. You should now see the Immich statistics being collected in Prometheus.

1. Import the included [grafana-dashboard.json](grafana-dashboard.json) to Grafana to visualize the statistics.