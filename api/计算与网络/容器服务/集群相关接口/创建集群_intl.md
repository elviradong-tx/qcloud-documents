## 1. API Description
This API (CreateCluster) is used to create clusters.
Domain name for API request: `ccs.api.qcloud.com`.

* When creating a cluster, you need to specify the number and configuration of nodes (CVMs) in the cluster.
* All nodes in the cluster are HDD cloud disks.
* By default, you can only create 5 clusters with an account. Apply for an increase in your quota by [submitting a ticket](https://console.cloud.tencent.com/workorder/category/create?level1_id=6&level2_id=350&level1_name=%E8%AE%A1%E7%AE%97%E4%B8%8E%E7%BD%91%E7%BB%9C&level2_name=%E5%AE%B9%E5%99%A8%E6%9C%8D%E5%8A%A1CCS).
* A cluster can only contain a maximum of 20 nodes. [Submit a ticket](https://console.cloud.tencent.com/workorder/category/create?level1_id=6&level2_id=350&level1_name=%E8%AE%A1%E7%AE%97%E4%B8%8E%E7%BD%91%E7%BB%9C&level2_name=%E5%AE%B9%E5%99%A8%E6%9C%8D%E5%8A%A1CCS) to request an increase in your quota, which should also be subject to the limit on the total number ([Learn more)](https://cloud.tencent.com/doc/product/213/CVM%E5%AE%9E%E4%BE%8B%E8%B4%AD%E4%B9%B0%E9%99%90%E5%88%B6).
* For more information on **limitations on the ratio** of CPU to memory, please see [here](https://cloud.tencent.com/doc/product/213/CVM%E5%AE%9E%E4%BE%8B%E9%85%8D%E7%BD%AE).
* After a node has been created, you can change the bandwidth using the API [UpdateInstanceBandwidthHour](https://cloud.tencent.com/doc/api/229/1345). **The public network bandwidth is 0 by default if not specified**.
* Supported node types **(the models purchased in each availability zone are different)**. For more information, please see [CVM Instance Configuration](https://cloud.tencent.com/doc/product/213/CVM%E5%AE%9E%E4%BE%8B%E9%85%8D%E7%BD%AE):


## 2. Input Parameters
The following request parameter list only provides API request parameters. Other parameters can be found in [Common Request Parameters](https://cloud.tencent.com/document/product/457/9463).

| Parameter Name | Description | Type | Required | 
|---------|---------|---------|---------|
| clusterName | Cluster name | String | Yes | 
| clusterDesc | Cluster description | String | No | 
| clusterCIDR| CIDR used to assign cluster containers and service IPs, which should not conflict with VPC CIDR or other cluster CIDRs within the same VPC | String | Yes | 
| ignoreClusterCIDRConflict | Whether to ignore ClusterCIDR conflict error. Default is 0.<br>0: Do not ignore (an error is returned)<br>1: Ignore (continue to create) | Int | No | 
| zoneId | Availability zone. Enter the Zone field returned via the API [Query Availability Zone](/doc/api/213/9455). | String | Yes |
| goodsNum | The number of nodes purchased. Maximum is 100. | Int | Yes | 
| cpu | Number of CPU cores. For more information on CPU limit, please see above. | Int | Yes |  
| mem| Memory size (in GB). For more information on memory limit, please see above. | Int | Yes | 
| osName| System name. Centos7.2x86_64 or ubuntu16.04.1 LTSx86_64, which is used by all nodes in the cluster and also automatically used for additional nodes. | String | Yes | 
| instanceType | For more information, please see [CVM Instance Configuration](https://cloud.tencent.com/doc/product/213/CVM%E5%AE%9E%E4%BE%8B%E9%85%8D%E7%BD%AE). Default value: S1.SMALL1 | String | No | 
| cvmType | Node type<br>PayByHour: Postpaid (default)<br>PayByMonth: Prepaid<br> | String | No | 
| renewFlag | Prepaid auto renewal flag. Value range:<br><li>NOTIFY_AND_AUTO_RENEW: Notify expiry and renew automatically<br><li>NOTIFY_AND_MANUAL_RENEW: Notify expiry but not renew automatically<br><li>DISABLE_NOTIFY_AND_MANUAL_RENEW: Neither notify expiry nor renew automatically<br><br>Default: NOTIFY_AND_AUTO_RENEW. If this parameter is specified as NOTIFY_AND_AUTO_RENEW, the instance is automatically renewed on a monthly basis upon its expiration when the account balance is sufficient. For more information, please see [InstanceChargePrepaid](https://cloud.tencent.com/document/api/213/9451#instancechargeprepaid). | String | No |
| bandwidthType | Bandwidth type<br>Prepaid CVMs: PayByMonth: Bill by bandwidth usage time. PayByTraffic: Bill by traffic<br>Postpaid CVMs: PayByHour: Bill by bandwidth usage time. PayByTraffic: Bill by traffic<br>For more information on the difference between the network billing methods, please see [Purchase Network Bandwidth](/doc/product/213/509). | String | Yes |
| bandwidth | Public network bandwidth (in Mbps), or the peak public network bandwidth when bandwidth type is "Bill-by-traffic" | Int | Yes | 
| wanIp | Whether to enable the public IP<br>0: Do not enable<br>1: Enable (default)<br>If bandwidth is greater than 0, you're free to choose whether to enable the public IP. If bandwidth is 0, the public IP is not assigned. | Int | No |
| vpcId | VPC ID. Enter the unVpcId (unified VPC ID) field returned via the API [Query VPC List](/doc/api/215/1372). | String | Yes |
| subnetId | Subnet ID. Enter the unSubnetId (unified subnet ID) field returned via the API [Query Subnet List](/doc/api/215/1371). | String | Yes | 
| isVpcGateway | Whether the [Public Gateway](/doc/product/215/3089#3.-.E5.90.91.E7.A7.81.E6.9C.89.E7.BD.91.E7.BB.9C.E4.B8.AD.E6.B7.BB.E5.8A.A0.E5.85.AC.E7.BD.91.E7.BD.91.E5.85.B3.) is used. The public gateway can be used only when it has a public IP and resides in VPC.<br>0: Non-public gateway<br>1: Public gateway | Int | Yes |  
| rootSize | System disk size. For Linux, the value range is 20 to 50 GB, and the increment is 1 GB. | Int | Yes | 
| rootType | Type of system disk. For more information on the limits of system disk type, please see [CVM Instance Configuration](https://cloud.tencent.com/doc/product/213/2177). Value range:<br><li>LOCAL_BASIC: Local disk<br><li>LOCAL_SSD: Local SSD disk<br><li>CLOUD_BASIC: HDD cloud disk<br><li>CLOUD_SSD: SSD cloud disk<br><br>Default: CLOUD_BASIC. | String | No |
| storageSize | Data disk size (in GB). The increment is 10. 0 means that no data disk is needed. For information on the maximum disk size, please see [Overview of Data Disk Products](/doc/product/213/498). | Int | Yes | 
| storageType | Type of data disk. For more information on the limits of data disk type, please see [CVM Instance Configuration](https://cloud.tencent.com/doc/product/213/2177). Value range:<br><li>LOCAL_BASIC: Local disk<br><li>LOCAL_SSD: Local SSD disk<br><li>CLOUD_BASIC: HDD cloud disk<br><li>CLOUD_PREMIUM: Premium cloud storage<br><li>CLOUD_SSD: SSD cloud disk<br><br>Default: CLOUD_BASIC.<br> | String | No |
| password | Node password. It is generated randomly if not set, and is sent via internal message. The node password must be a combination of 8-16 characters comprised of at least two of the following types: [a-z, A-Z], [0-9] and [( ) & # 96; ~ ! @ # $ % ^ & * - + = & #124; { } [ ] : ; ' < >, . ? / ] | String | No | 
| keyId | [Key](/doc/product/213/503) ID. You can use the key to log in to the node after the key is associated. "keyId" can be obtained via the API [Query Key](/doc/api/229/%E6%9F%A5%E8%AF%A2%E5%AF%86%E9%92%A5). Key and password cannot be specified at the same time, and specifying key is not allowed in Windows. | String | No |
| period | Usage period purchased for a prepaid node (in month). When cvmType is PayByMonth, this parameter is required. | Int | No | 
| sgId | Security group ID. No security group is associated by default. Enter the sgId field returned via the API [Query Security Group List](/doc/api/213/1232). | String | No | 
| masterSubnetId | Cluster master occupies the IP of a VPC subnet. This parameter specifies the subnet in which the IP occupied by Master resides. The subnet must be located in the same VPC as the cluster. | String | No | 

## 3. Output Parameters
 
| Parameter Name | Description | Type |
|---------|---------|---------|
| code | Common error code. 0: Successful; other values: Failed. For more information, please see [Returned Failure Result](/doc/api/457/9469). | Int | 
| codeDesc | Error code at business side. For a successful operation, "Success" is returned. In case of an error, a message describing the reason for the error is returned. | String |
| message | Module error message description depending on API. For more information, please see [Returned Failure Result](/doc/api/457/9469). | String | 
| requestId | Task ID | Int |
| clusterId | Cluster ID | Int |

## 4. Example
Input
```
  https://domain/v2/index.php?Action=CreateCluster
  &imageId=1
  &bandwidth=1
  &cpu=1
  &mem=2
  &storageType=1
  &storageSize=50
  &goodsNum=1
  &zoneId=1
  &vpcId=vpc-8e0ypm3z 
  &subnetId=subnet-3lzrkspo 
  &other common parameters
```
Output
```
  {
      "code" : 0,
      "message" : "ok",
      "codeDesc": "Success",
      "data":{
	      "clusterId":"cluster-xxxx",
	      "requestId":11333
      }
      
  }

```

