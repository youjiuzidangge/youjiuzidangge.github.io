---
layout: post
title:  利用 jquery 对 div 进行排序
#时间配置
date:   2017-05-06 16:30:00 +0800
#大类配置
categories: 代码
#小类配置
tag: jquery
---

* content
{:toc}


	需求：需要对标签排序的情况。
	这个例子很有趣，是根据子元素标签对父元素标签进行排序。
	
----------------------------------
	<div class='container'>
		<div id='1'>
			<div class='name'>bbbb</div>
			<div class='number'><input type='text' value='110'/></div>
		</div>
		<div id='2'>
			<div class='name'>cccc</div>
			<div class='number'><input type='text' value='120'/></div>
		</div>
		<div id='3'>
			<div class='name'>dddd</div>
			<div class='number'><input type='text' value='140'/></div>
		</div>
	</div>

	<button id="desc">从大到小</button>
	<button id="asc">从小到大</button
	
	
	$(function() {
		var asc = function(a, b) {
			return $(a).find('input').val() > $(b).find('input').val() ? 1 : -1;
		}

		var desc = function(a, b) {
			return $(a).find('input').val() > $(b).find('input').val() ? -1 : 1;
		}

		var sortByInput = function(sortBy) {
			var sortEle = $('.container>div').sort(sortBy);
			$('.container').empty().append(sortEle);
		}

		$('#desc').click(function() {
			sortByInput(desc);
		});

		$('#asc').click(function() {
			sortByInput(asc);
		});
	})
 