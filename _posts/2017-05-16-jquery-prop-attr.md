---
layout: post
title:  jquery��attr��prop������
#ʱ������
date:   2017-05-18 22:06:00 +0800
#��������
categories: ����
#С������
tag: jquery
---

* content
{:toc}

�������ø��� input ��ѡ״̬ʱ����ô���Ĳ��ˣ���������������� prop ������һ�Լ��顣�����Ž�Լ�Ͷ��ɱ���ʡʱ��ԭ��
�ҾͲ��Լ�һ���д����ת����[������ͷ���ļ����ռ�]http://www.cnblogs.com/Showshare/p/different-between-attr-and-prop.html) 

--------------------------------------------------

#jquery��attr��prop������

�ڸ߰汾��jquery����prop������ʲôʱ�����prop��ʲôʱ����attr����������֮����ʲô������Щ����ͳ����ˡ�

���������������������ϵĴ𰸺ܶࡣ����̸̸�ҵ��ĵã��ҵ��ĵúܼ򵥣�

+����HTMLԪ�ر���ʹ��еĹ������ԣ��ڴ���ʱ��ʹ��prop������
+����HTMLԪ�������Լ��Զ����DOM���ԣ��ڴ���ʱ��ʹ��attr������
 

���������Ҳ���е�ģ�����ټ������Ӿ�֪���ˡ� 

		<a href="http://www.baidu.com" target="_self" class="btn">�ٶ�</a>
		
���������<a>Ԫ�ص�DOM�����С�href��target��class"����Щ���Ծ���<a>Ԫ�ر���ʹ��е����ԣ�Ҳ��W3C��׼��Ͱ������⼸�����ԣ�
����˵��IDE���ܹ�������ʾ�������ԣ���Щ�ͽ����������ԡ�������Щ����ʱ������ʹ��prop������

		<a href="#" id="link1" action="delete">ɾ��</a>

���������<a>Ԫ�ص�DOM�����С�href��id��action���������ԣ�ǰ�����ǹ������ԣ�������һ����action�������������Լ��Զ�����ȥ�ģ�
<a>Ԫ�ر�����û��������Եġ����־����Զ����DOM���ԡ�������Щ����ʱ������ʹ��attr������ʹ��prop����ȡֵ����������ֵʱ��
���᷵��undefinedֵ��

�پ�һ�����ӣ�

		<input id="chk1" type="checkbox" />�Ƿ�ɼ�
		<input id="chk2" type="checkbox" checked="checked" />�Ƿ�ɼ�

��checkbox��radio��select������Ԫ�أ�ѡ�����Զ�Ӧ��checked���͡�selected������ЩҲ���ڹ������ԣ������Ҫʹ��prop����ȥ������
�ܻ����ȷ�Ľ����

		$("#chk1").prop("checked") == false
		$("#chk2").prop("checked") == true
		
�������ʹ��attr�����������֣�

		$("#chk1").attr("checked") == undefined
		$("#chk2").attr("checked") == "checked"
		
ȫ���ꡣ
	