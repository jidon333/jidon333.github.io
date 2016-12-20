---
layout: post
title: This is my first post Cool!
categories: [TestCate]
comments: true
tags: [TestTag]
---

# Hello My first post

- 수학
- C/C++
- 컴퓨터 그래픽스

**English**


// some c# code	
var a = "bad variable name"



이거 한글 가능함???

ㅋ

이미지 올리기 첫번째 방법

괄호안에 URL을 넣는다.
![my github profile image!](https://avatars2.githubusercontent.com/u/16080882?v=3&s=460)


이미지 올리기 두번째  방법

Assets folder에 직접 넣고 경로를 지정해준다.
![Thinking cat](/assets/img/ThinkingCat.jpg)



{% if page.comments %}
	<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = '//jidon333.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();

</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
                                
{% endif %}

<script id="dsq-count-scr" src="//jidon333.disqus.com/count.js" async></script>
