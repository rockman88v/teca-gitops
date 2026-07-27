# Cài đặt ELK stack trên k8s bằng helmchart

## Chuẩn bị
**Lấy thông tin storageclass trên k8s**
```bash
kubectl get sc
```
*Lưu ý: Update các file helm-value sử dụng đúng storageclass này*

**Tạo namespace cho logging và đặt làm ns mặc định**
```bash
kubectl create ns logging
kn logging
```
**Giải nén các helmchart của bộ ELK**
```bash
cd devops_kubernetes/logging/elk-stack/
tar -xzf filebeat-7.17.3.tgz
tar -xzf elasticsearch-7.17.3.tgz
tar -xzf kibana-7.17.3.tgz
tar -xzf logstash-7.17.3.tgz
```
Kết quả có 4 thư mục được tạo ra:
```bash
sysadmin@master:~/devops_kubernetes/logging/elk-stack$ ls -l |grep -v tgz
total 92
drwxrwxr-x 4 sysadmin sysadmin  4096 Jan  5 09:18 elasticsearch
drwxrwxr-x 4 sysadmin sysadmin  4096 Jan  5 09:17 filebeat
drwxrwxr-x 4 sysadmin sysadmin  4096 Jan  5 09:18 kibana
drwxrwxr-x 4 sysadmin sysadmin  4096 Jan  5 09:18 logstash
-rw-rw-r-- 1 sysadmin sysadmin  1286 Jan  5 09:06 README.md
```

## Cài đặt ứng dụng lên k8s 
*Mặc định cài ở ns `logging`, ns này đã được tạo và set làm mặc định ở bước trên*
### Cài đặt ElasticSearch
*Sử dụng file `value-elasticsearch.yaml`. Lưu ý cập nhật thông tin storageclass trong file đúng với storageclass của bạn đã check ở bước trên*

Cài đặt ElasticSearch:
```bash
helm install elasticsearch -f value-elasticsearch.yaml ./elasticsearch
```
<aside>
💡 Sau khi cài đặt, mỗi khi chúng ta muốn tùy biến các tham số của ứng dụng bằng cách cập nhật file value thì chúng ta update vào hệ thống bằng lệnh sau:

</aside>

**Lệnh upgrade helm-chart release:**

```bash
# update release
helm upgrade elasticsearch -f value-elasticsearch.yaml ./elasticsearch
```

<aside>
💡 Sau khi ElasticSearch được cài đặt xong (các Pod đều ở trạng thái Running) thì ta có thể kết nối vào để kiểm tra thông qua [worker-node-PublicIP]:[NodePort]. Ở đây đang set NodePort là 30092
</aside>

### Cài đặt logstash
*Sử dụng file `value-logstash.yaml`. Lưu ý cập nhật thông tin storageclass trong file đúng với storageclass của bạn đã check ở bước trên*

Cài đặt logstash:
```bash
helm install logstash -f value-logstash.yaml ./logstash
```
<aside>
💡 Sau khi cài đặt, mỗi khi chúng ta muốn tùy biến các tham số của ứng dụng bằng cách cập nhật file value thì chúng ta update vào hệ thống bằng lệnh sau:

</aside>

**Lệnh upgrade helm-chart release:**

```bash
# update release
helm upgrade logstash -f value-logstash.yaml ./logstash
```

### Cài đặt filebeat
*Sử dụng file `value-filebeat.yaml`. Lưu ý cập nhật thông tin storageclass trong file đúng với storageclass của bạn đã check ở bước trên*

Cài đặt filebeat:
```bash
helm install filebeat -f value-filebeat.yaml ./filebeat
```
<aside>
💡 Sau khi cài đặt, mỗi khi chúng ta muốn tùy biến các tham số của ứng dụng bằng cách cập nhật file value thì chúng ta update vào hệ thống bằng lệnh sau:

</aside>

**Lệnh upgrade helm-chart release:**

```bash
# update release
helm upgrade filebeat -f value-filebeat.yaml ./filebeat
```
### Cài đặt Kibana
*Sử dụng file `value-kibana.yaml`. Lưu ý cập nhật thông tin storageclass trong file đúng với storageclass của bạn đã check ở bước trên*

Cài đặt kibana:
```bash
helm install kibana -f value-kibana.yaml ./kibana
```
<aside>
💡 Sau khi cài đặt, mỗi khi chúng ta muốn tùy biến các tham số của ứng dụng bằng cách cập nhật file value thì chúng ta update vào hệ thống bằng lệnh sau:

</aside>

**Lệnh upgrade helm-chart release:**

```bash
# update release
helm upgrade kibana -f value-kibana.yaml ./kibana
```

<aside>
💡 Sau khi Kibana được cài đặt xong (các Pod đều ở trạng thái Running) thì ta có thể kết nối vào để kiểm tra thông qua [worker-node-PublicIP]:[NodePort]. Ở đây đang set NodePort là 30093
</aside>

### Cập nhật logstash sử dụng grok
*Tham khảo trang debug cấu hình grok online: https://grokdebugger.com/*

Khi xem log của ứng dụng `product-app` ta thấy log nằm ở fields `message` có format như sau:
```
INFO:product:10.233.125.4 - - [05/Jan/2025 09:38:22] "GET /promotion HTTP/None" 200 - 784.51ms
```
Trong đó có chứa một số thông tin quan trọng như `method`, `apiPath`, `responseCode` và `responseTime` nhưng chúng ta chỉ có thể tìm kiếm dạng keyword.

Bài toán đặt là chúng ta muốn filter theo các tham số đó thì làm thế nào?

Lúc này cần cấu hình thêm phần xử lý dữ liệu ở logstash trước khi đẩy về ElasticSearch

**Update lại logstash**

```bash
# update release
helm upgrade logstash -f value-logstash.yaml -f value-logstash-grok.yaml ./logstash
```
Lúc này filter logs của ứng dụng `product-app` ta sẽ thấy logs của nó được phân tích và có các fields mới như `method`, `apiPath`, `responseCode` và `responseTime`.