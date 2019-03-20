# 开源一个用于查看 Bandwagon  VPS 流量使用情况的小网页（PHP）
@[toc]
## 前言
 Bandwagon 的额外功能里提供了一些 API，可以用于控制 VPS 的开关机 / 备份 / 迁移等，也能用于查询 VPS 的众多信息，如计划详情 / 结点信息 / 流量用量等。想着平时登录控制面板查个流量挺费劲的，就打算用 Bandwagon 提供的 API 写个专门用于查看 Bandwagon  VPS 流量使用情况的小网页。

至于查看实时 CPU 利用率，实时 RAM 占用这些... 一来没啥大用，二来获取这些信息最简单的方式莫过于跑个 SHELL，但这由于涉及安全问题而略有些麻烦。索性，我就懒得实现了。

实现过程中最大的坑莫过于时区的问题，VPS 和 控制面板默认是 **EDT** 时间，但是我用来配置环境的 LNMP 包似乎给我替换成 **PRC** 了。API 返回的，表示流量重置时间的 UNIX 时间戳，是以美国时间为准的，所以我也得做相应修改。

然后我也懒得去琢磨 Bandwagon 每个流量重置周期究竟是 30 天，还是根据新的周期开始时所在的月份实时计算的。图省事儿，流量重置进度的百分比，我是通过计算流量重置剩余时间占 30 天的百分比来计算的。**因此，流量重置进度的 百分比 可能有误，不过流量重置的 具体时间 无误。**

另外，部分计划有 **流量倍乘** 的概念，如 0.33x。 这是说返回的统计结果中，使用的流量和计划的流量要同乘 0.33。比如返回结果显示，我的每月流量为 550 GiB，现已使用 0.48 GiB，控制面板就应显示每月流量为 183 Gib，现已使用 0.16 GiB。不管流量在 Bandwagon 内部是如何统计的，反正**就显示而言，我的代码是没问题的。**

使用说明见代码头部注释。
## 效果图（1080P）
![统计页面图](http://img.blog.csdn.net/20170926212023755?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFva2Vsb25nOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![控制面板图](http://img.blog.csdn.net/20170926211959594?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFva2Vsb25nOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 源代码
```
<?php
      /**
      * 说明书
      * 第一步，在服务器上安装 LNMP https://lnmp.org/install.html
      * 第二步，将修改后的本文件命名为 index.php，并复制入虚拟主机默认目录 /home/wwwroot/default
      * 第三步，删除原目录中的 index.html
      * 第四步，通过主机 IP 访问该站点
      *
      * 猫科龙
      * 2017-09-26 
      */

      // ** 修改这里的配置信息 **
      $VEID = 'xxxxxx';
      $API_KEY = 'private_xxxxxxxxxxxxxxxxxxxxxx';
      // *************************************

      // 申请统计信息
      $request_URL = 'https://api.64clouds.com/v1/getServiceInfo?veid=' . $VEID . '&api_key=' . $API_KEY;
      $serviceInfo = json_decode (file_get_contents ($request_URL));

      // 反序列化统计信息
      $ip_addresses = $serviceInfo->ip_addresses;
      $node_location = $serviceInfo->node_location;

      $data_counter = floatval($serviceInfo->data_counter);
      $plan_monthly_data = floatval($serviceInfo->plan_monthly_data);
      $monthly_data_multiplier = floatval($serviceInfo->monthly_data_multiplier);

      $data_next_reset = intval($serviceInfo->data_next_reset) + 24 * 60 * 60;
      
      // 加工统计信息
      $ipstr = '';
      foreach($ip_addresses as $key => $value) {
            $ipstr .= $value . ' ';
      }
      
      $data_used = round($data_counter * $monthly_data_multiplier / pow(2, 30), 2);
      $data_total = round($plan_monthly_data * $monthly_data_multiplier / pow(2, 30), 2);
      $data_usage = round($data_used / $data_total * 100, 2);

      $data_cycle_now = date("Y-m-d H:i:s", time());
      $data_cycle_reset = date("Y-m-d H:i:s", $data_next_reset);
      $data_cycle_remainder = 100 - round(($data_next_reset - time()) / (30 * 24 * 60 * 60) * 100, 2);
      $sn = 0;
?>
<!DOCTYPE html>
<html lang="en">
<head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta http-equiv="X-UA-Compatible" content="ie=edge">
      <title>BandwagonHost VPS Data Counter</title>
      <meta charset="UTF-8">
      <link rel="stylesheet" href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u"
            crossorigin="anonymous">
      <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js" integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa"
            crossorigin="anonymous"></script>
</head> 
<body>
      <div class="container">

            <div class="page-header">
                  <h1> BandwagonHost VPS Data Counter <small>  for <?php echo $ipstr; ?> in <?php echo $node_location;  ?> </small></h1>
            </div>

            <div class="panel panel-default">
                  <div class="panel-heading">
                        <h3 class="panel-title">Main Panel</h3>
                  </div>
                  <div class="panel-body">
                        <table class="table table-default">
                              <thead>
                                    <!-- header of the table -->
                                    <tr>
                                          <th>SN</th>
                                          <th>Intro</th>
                                          <th>Detail</th>
                                    </tr>
                              </thead>
                              <tbody>
                                    <!-- data usage -->
                                    <tr>
                                          <td rowspan="2"><?php echo $sn++; ?></td>
                                          <td rowspan="2">Data Usage</td>
                                          <td><?php echo $data_used . ' GiB / ' . $data_total . ' GiB'; ?></td>
                                    </tr>
                                    <tr>
                                          <td>
                                                <div class="progress">
                                                      <div class="progress-bar progress-bar-success" role="progressbar" aria-valuenow="<?php echo $data_usage;?>" aria-valuemin="0" aria-valuemax="100" style="width: <?php echo $data_usage;?>%;">
                                                            <span class="sr-only"><?php echo $data_usage . '%';?> </span>
                                                            <?php if($data_usage >= 50) echo $data_usage . ' %';?>
                                                      </div>
                                                      <?php if($data_usage < 50) echo $data_usage . ' %';?>
                                                </div>
                                          </td>
                                    </tr>

                                    <!-- data reset cycle -->
                                    <tr>
                                          <td rowspan="2"><?php echo $sn++; ?></td>
                                          <td rowspan="2">Data Reset Cycle</td>
                                          <td><?php echo $data_cycle_now . ' / ' . $data_cycle_reset . ' ' .  date_default_timezone_get(); ?></td>
                                    </tr>
                                    <tr>
                                          <td>
                                                <div class="progress">
                                                      <div class="progress-bar progress-bar-success" role="progressbar" aria-valuenow="<?php echo $data_cycle_remainder;?>" aria-valuemin="0" aria-valuemax="100" style="width: <?php echo $data_cycle_remainder;?>%">
                                                            <span class="sr-only"><?php echo $data_cycle_remainder;?>% </span>
                                                            <?php if($data_cycle_remainder >= 50) echo 'About ' . $data_cycle_remainder . ' %'?>
                                                      </div>
                                                      <?php if($data_cycle_remainder < 50) echo 'About ' . $data_cycle_remainder . ' %'?>
                                                </div>
                                          </td>
                                    </tr>
                              </tbody>
                        </table>
                  </div>
            </div>

      </div>

</body>

</html>
```
