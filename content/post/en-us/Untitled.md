#英特尔软件和数据中心版固态硬盘的安全漏洞

![](https://img.purch.com/intel-ssd-dc-s4500/w/755/aHR0cDovL21lZGlhLmJlc3RvZm1pY3JvLmNvbS8yL1QvODQ1MDkzL29yaWdpbmFsL2ludGVsLXNzZC1kYy5KUEc=)
> Intel SSD DC S4500. Credit: Intel

自从2018年被发现熔断(Meltdown)和幽灵(Spectre)漏洞以来，英特尔一直是安全研究的焦点。
安全研究人员最近发现英特尔软件的新缺陷，其中包括来自Eclypsium的安全研究员Jesse Michael发现的处理器诊断工具的高危漏洞，以及英特尔工程师发现的数据中心版本SSD中的一个漏洞。

#新发现的两个英特尔软件漏洞
处理器诊断工具（CVE-2019-11133）中的缺陷在CVSS 3.0规模上评为8.2（满分10分），这使其成为一个高危漏洞。根据英特尔最新的安全公告，该漏洞“可能允许经过身份验证的用户做到本地提权，信息泄露或
拒绝服务”。低于4.1.2.24版本会受到影响。

英特尔内部团队发现的第二个漏洞是销售给数据中心客户的英特尔SSD DC S4500 / S4600系列中的中等严重性漏洞。低于版本SCV10150的SSD固件版本中发现的缺陷在CVSS 3.0规模上获得了5.3分，因此它被标记为中等严重性。该错误可能允许非特权用户通过物理访问启用权限提升。

由于其中一个缺陷是由英特尔本身发现的，另一个是与英特尔协调的Eclypsium研究披露，英特尔能够及时准备好这些补丁以供公众宣布。


sed -i 's/ZSH_THEME=\"robbyrussell\"/ZSH_THEME=\"agnoster\"/g' ~/.zshrc