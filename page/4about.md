---
layout: page
title: About
permalink: /about/
icon: heart
type: page
---

## 关于我

蒋李恒，男，80后，软件测试从业者。

**工作历程**

10年软件测试经验，精通各种测试方法和工具。参与过的项目有多个金融机构的核心系统建设、某金融机构链路预警大数据分析项目、某支付公司的支付系统搭建、测试工具平台咨询实施、CI&CD 咨询实施、DevOps体系建设等。

也曾短暂转岗过 Java大数据开发（1年半左右）、CI&CD&DevOps（2家公司）、项目管理（6个项目）、产品经理（2个项目）等非测试工作。

**目前情况**

现在就职于一家金融科技创业公司，从事DevOps、质量管理相关工作。


**微信公众号**

![](/assets/images/about/daodaotest.jpg)

## 联系我

- Github: {% if site.github_username %}<a href="https://github.com/{{ site.github_username }}" title="github_username">{{ site.github_username }}</a> {% endif %}
- Email: {% if site.email %}<a href="mailto:{{ site.email }}" title="email">{{ site.email }}</a> {% endif %}
- Wechat: crarook

## 关于本博

本博采用 Jekyll[[1]][1] 搭建，Markdown[[2]][2] 写作，托管于 GitHub[[3]][3]。

自 2020 年 03 月 07 日起，本站已运行 <span id="days"></span> 天，截至 {{ site.time | date: "%Y 年 %m 月 %d 日" }}，写了博文 {{ site.posts.size }} 篇，{% assign count = 0 %}{% for post in site.posts %}{% assign single_count = post.content | strip_html | strip_newlines | remove: ' ' | size %}{% assign count = count | plus: single_count %}{% endfor %}{% if count > 10000 %}{{ count | divided_by: 10000 }} 万 {{ count | modulo: 10000 }}{% else %}{{ count }}{% endif %} 字。

即日起，本博客的原创内容，均采用知识共享组织（Creative Commons）的 "署名-非商业性使用 3.0 中国大陆 (CC BY-NC 3.0 CN)[[4]][4]" 许可。

## 赞赏奖励

若您觉得本博客所创造的内容对您有所帮助，可考虑略表心意，支持一下。

{% include reward.html %}

[1]: https://jekyllrb.com/ 'Jekyll'
[2]: http://daringfireball.net/projects/markdown/ 'Markdown'
[3]: https://github.com/ 'GitHub'
[4]: http://creativecommons.org/licenses/by-nc/3.0/cn/ '署名-非商业性使用 3.0 中国大陆'

{% include comments.html %}

<script>
var days = 0, daysMax = Math.floor((Date.now() / 1000 - {{ "2020-03-07" | date: "%s" }}) / (60 * 60 * 24));
(function daysCount(){
    if(days > daysMax){
        document.getElementById('days').innerHTML = daysMax;
        return;
    } else {
        document.getElementById('days').innerHTML = days;
        days += 10;
        setTimeout(daysCount, 1);
    }
})();
</script>
