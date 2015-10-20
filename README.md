# cloudwatch-munin-node

It may be convenient if put the Custom Metrics directly AWS CloudWatch from munin-node. As study tried to create in casual.

## Motivation

Munin is a convenient, and can easy installation. But the monitoring point is increased, load of RRD processing or graph generation process will be a problem, in munin-server. AWS has a service called CloudWatch, it can accumulate and visualization and monitoring of metric data. More recently, the Custom Metrics, it is possible to put your own data to CloudWatch. Therefore, be to put your own data directly to the CloudWatch from each munin-node, you will be able to take over the munin-server.

However CloudWatch can not specify the detailed data type and drawing. Use of this script is the extent such use is in a limiting metric item.

## BUG
In basic the Unit was 'None'. If 'upper-limit' exist, was 'Percent'. Other decision was not implemented.

Very simple implements in cdef.

# Installation

* install munin-node

<pre>
sudo aptitude install munin-node
sudo aptitude install munin-plugins-extra
</pre>

* [From this site](http://effbot.org/zone/socket-intro.htm) Make a sample source of 'python socket' from SimpleClient.py.

* [loggly / loggly-watch](https://github.com/loggly/loggly-watch) Get cloudwacth.py, apply patch.

<pre>
--- cloudwatch.py.orig	2011-06-16 13:31:38.000000000 +0900
+++ cloudwatch.py	2011-06-16 08:31:11.000000000 +0900
@@ -38,7 +38,15 @@
         self.key = os.getenv('AWS_ACCESS_KEY_ID', key)
         self.secret_key = os.getenv('AWS_SECRET_ACCESS_KEY_ID', secret_key)

-    def putData(self, namespace='Loggly', metricname='EventCount', value=0):
-        foo = getSignedURL(self.key, self.secret_key, 'PutMetricData', {'Namespace': namespace, 'MetricData.member.1.MetricName': metricname, 'MetricData.member.1.Value': value})
+    def putData(self, namespace='Loggly', dimensionsname='InstanceId', dimensionsvalue='MyInstanceId',
+                metricname='EventCount', unit='None', value=0.0):
+        foo = getSignedURL(self.key, self.secret_key, 'PutMetricData', {
+                'Namespace': namespace,
+                'MetricData.member.1.Dimensions.member.1.Name': dimensionsname,
+                'MetricData.member.1.Dimensions.member.1.Value': dimensionsvalue,
+                'MetricData.member.1.MetricName': metricname,
+                'MetricData.member.1.Unit': unit,
+                'MetricData.member.1.Value': value})
         h = httplib2.Http()
         resp, content = h.request(foo)
+        # print content
</pre>

* Place cloudwatch-munin-node.py, SimpleClient.py, cloudwatch.py in the application directory.

* Specify what you need within the metric item of muninin, in an array 'QLIST' of cloudwatch-munin-node.py.

Caution:　CloudWatch will be charged by AWS. Try carefully.

* Put your 'AWS Access Key ID' in AWS_ACCESS_KEY_ID. and put your 'AWS Secret Access Key' in AWS_SECRET_ACCESS_KEY.

Note: Use IAM. will be (relatively) safe.

Sample of CloudWatch Policy.
<pre>
{
  "Statement": [
    {
      "Action": "cloudwatch:*",
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
</pre>

* Set cron job.

<pre>
crontab -e
4-59/5 * * * * /SOME/WHERE/cloudwatch-munin-node.py
</pre>
