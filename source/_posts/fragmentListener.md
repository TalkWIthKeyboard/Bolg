---
title: fragment监听事件
date: 2016-04-10 19:56:26
categories: Android
tags: 77
comments: true

---
### searchView控件为例
在用Fragment做菜单切换，不再像Activity实例一样可以监听页面上所有物理键的事件，那这种问题如何解决呢？
采用new一个新的监听实例的方法来解决(下图以SearchView监听事件为例，提交查询后跳转页面)：

	import android.support.v4.app.Fragment;
	import android.os.Bundle;
	import android.view.LayoutInflater;
	import android.view.View;
	import android.view.ViewGroup;
	import android.widget.SearchView;
	import android.widget.SearchView.OnQueryTextListener;
	import android.content.Intent;
	
	
	
	public class SearchFragment extends Fragment {
	SearchView sv = null;
	View view;
	
	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
	Bundle savedInstanceState) {
	view = inflater.inflate(R.layout.fragment_search, container, false);
	sv = (SearchView) view.findViewById(R.id.sv);
	sv.setIconifiedByDefault(false);
	sv.setSubmitButtonEnabled(true);
	sv.setQueryHint("姓名/学校/专业");
	
	/*利用searchView创建一个监听对象实例并重写其方法 */
	sv.setOnQueryTextListener(new OnQueryTextListener(){
	
	@Override
	public boolean onQueryTextSubmit(String submit) {
	/*fragment向activity跳转*/
	Intent intent = new Intent(getActivity(), resultActivity.class);
	startActivity(intent);
	return true;
	}
	
	@Override
	public boolean onQueryTextChange(String change) {
	
	return false;
	}
	});
	
	return view;
	}

