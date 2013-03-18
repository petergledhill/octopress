---
layout: post
title: "Remove ASPNETDB.mdf for MVC sites using SimpleMembership"
author: Peter Gledhill
date: 2013-03-18 16:05
comments: true
categories: [C#, MVC4]
---

If you create a new MVC4 application you will notice a .MDF in your App_Data folder.

If you use a connection string to a database provider like MSSQL or MYSQL, you might wonder why this file is needed. And indeed why it comes back every time you delete it!  

The file is created by the AspNetSqlMembershipProvider and AspNetSqlRoleProvider.  However, your MVC4 application will be using SimpleMembership / SimpleRole providers which don't require this file. 

The problem is that AspNetSqlMembershipProvider will be specified in your machine.config which is inherited by your web.config (mines found in C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config).  

I think there's a bit of magic going on here: the old providers are loaded first, then the SimpleProviders override them.

## How to remove the ASPNETDB.mdf dependency

In your web.config: clear the connection strings inherited from machine.config

{% codeblock Web.config lang:xml %}
 <connectionStrings>   
	<clear/>
 	<add name="DefaultConnection" connectionString="Data Source=localhost\SQLEXPRESS;Initial Catalog=test;Integrated Security=True;Persist Security Info=False;MultipleActiveResultSets=True;Packet Size=4096" providerName="System.Data.SqlClient" />
  </connectionStrings>
{% endcodeblock %}

In your web.config: explicitly set the providers to be SimpleProviders and clear out any inherited providers.

{% codeblock Web.config lang:xml %}
 <system.web>      
    <roleManager enabled="true" defaultProvider="SimpleRoleProvider">
      <providers>
        <clear />
        <add name="SimpleRoleProvider" type="WebMatrix.WebData.SimpleRoleProvider, WebMatrix.WebData" />
      </providers>
    </roleManager>
    <membership defaultProvider="SimpleMembershipProvider">
      <providers>
        <clear />
        <add name="SimpleMembershipProvider" type="WebMatrix.WebData.SimpleMembershipProvider, WebMatrix.WebData" />
      </providers>
    </membership>
</system.web>
{% endcodeblock %}

You can now remove the .mdf and .ldf files from your App_Data folder and they shouldn't return.

### Why do this?

I originally found out how to do this because I was having a problem on [Appharbour](http://www.appharbour.com).  Some of my user data was getting stored in this database even though I had defined a MSSQL connection string.  

However I can't recreate the problem and this was probably not the cause of the issue.  Even so, it's nice to be sure!