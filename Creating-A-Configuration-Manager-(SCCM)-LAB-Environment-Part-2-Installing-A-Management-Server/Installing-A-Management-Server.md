---


---

<p>In these series of posts, we will go through the steps required to install Configuration Manager in a simple LAB environment. The LAB environment will be referenced in future posts as we explore Configuration Manager further. See  <a href="../Creating-A-Configuration-Manager-(SCCM)-LAB-Environment-Part-1-Installing-Active-Directory/Installing-Active-Directory.md">Part 1</a>  for an overview of the LAB environment.</p>
<p>Last time we got our Domain Controller up and running and in this post we will install a Management server for administrations purposes. This is an optional step in the series, but it makes life a lot easier having a single server with all the required management tools. LABADM01 should be used for this purpose and should only have the operating system installed.</p>
<p>Quick Jump:<br>
<a href="../Creating-A-Configuration-Manager-(SCCM)-LAB-Environment-Part-1-Installing-Active-Directory/Installing-Active-Directory.md">Part 1 – Overview and Domain Controller installation</a><br>
<a href="../Creating-A-Configuration-Manager-(SCCM)-LAB-Environment-Part-2-Installing-A-Management-Server/Installing-A-Management-Server.md">Part 2 – Management Server Installation</a><br>
<a href="../Creating-A-Configuration-Manager-(SCCM)-LAB-Environment-Part-3-Installing-SQL-Server/Installing-SQL-Server.md">Part 3 – Installing SQL Server</a><br>
<a href="../Creating-A-Configuration-Manager-(SCCM)-LAB-Environment-Part-4-Configuration-Manager-Prerequisites/Configuration-Manager-Prerequisites.md">Part 4 – Configuration Manager Prerequisites</a><br>
<a href="../Creating-A-Configuration-Manager-(SCCM)-LAB-Environment-Part-5-Installing-Configuration-Manager/Installing-Configuration-Manager.md">Part 5 – Installing Configuration Manager</a></p>
<h3 id="basic-server-configuration">Basic Server Configuration</h3>
<p>Since setting up the management server is rather straight forward and an optional component in the series, we will automate the process with PowerShell. Like before we will setup the networking then install the required management tools.</p>
<p>Issue the commands below to set the computers IP address or specify it manually.</p>
<pre class=" language-powershell"><code class="prism  language-powershell"><span class="token function">Rename-Computer</span> <span class="token operator">-</span>NewName LABADM01
<span class="token function">Write-Host</span> <span class="token string">"Computer Name Changed"</span>
<span class="token variable">$AdminKey</span> = <span class="token string">"HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}"</span>
<span class="token variable">$UserKey</span> = <span class="token string">"HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A8-37EF-4b3f-8CFC-4F3A74704073}"</span>
<span class="token function">Set-ItemProperty</span> <span class="token operator">-</span>Path <span class="token variable">$AdminKey</span> <span class="token operator">-</span>Name <span class="token string">"IsInstalled"</span> <span class="token operator">-</span>Value 0
<span class="token function">Set-ItemProperty</span> <span class="token operator">-</span>Path <span class="token variable">$UserKey</span> <span class="token operator">-</span>Name <span class="token string">"IsInstalled"</span> <span class="token operator">-</span>Value 0
<span class="token function">Write-Host</span> <span class="token string">"Disabled IE Enhanced Security Configuration"</span>
New<span class="token operator">-</span>NetIPAddress <span class="token operator">-</span>InterfaceAlias <span class="token string">"Ethernet"</span> <span class="token operator">-</span>AddressFamily IPv4 <span class="token operator">-</span>IPAddress 192<span class="token punctuation">.</span>168<span class="token punctuation">.</span>3<span class="token punctuation">.</span>30 <span class="token operator">-</span>PrefixLength 24 <span class="token operator">-</span>DefaultGateway 192<span class="token punctuation">.</span>168<span class="token punctuation">.</span>3<span class="token punctuation">.</span>1
<span class="token function">Set</span><span class="token operator">-</span>DnsClientServerAddress <span class="token operator">-</span>InterfaceAlias <span class="token string">"Ethernet"</span> <span class="token operator">-</span>ServerAddresses 192<span class="token punctuation">.</span>168<span class="token punctuation">.</span>3<span class="token punctuation">.</span>20
<span class="token function">Write-Host</span> <span class="token string">"IP address and DNS set"</span>
<span class="token function">Set-ItemProperty</span> <span class="token operator">-</span>Path <span class="token string">"HKLM:\System\CurrentControlSet\Control\Terminal Server"</span> <span class="token operator">-</span>Name <span class="token string">"fDenyTSConnections"</span> –Value 0
Enable<span class="token operator">-</span>NetFirewallRule <span class="token operator">-</span>DisplayGroup <span class="token string">"Remote Desktop"</span>
<span class="token function">Write-Host</span> <span class="token string">"Enabled Remote Desktop"</span>
<span class="token function">Restart-Computer</span>
</code></pre>
<p>Next, issue the following command to install all the required management tools:</p>
<pre class=" language-powershell"><code class="prism  language-powershell">Install<span class="token operator">-</span>WindowsFeature <span class="token operator">-</span>Name RSAT<span class="token operator">-</span>ADDS<span class="token punctuation">,</span> RSAT<span class="token operator">-</span>ADDS<span class="token operator">-</span>Tools<span class="token punctuation">,</span> RSAT<span class="token operator">-</span>AD<span class="token operator">-</span>Powershell<span class="token punctuation">,</span> RSAT<span class="token operator">-</span>DHCP<span class="token punctuation">,</span> RSAT<span class="token operator">-</span>DNS<span class="token operator">-</span>Server<span class="token punctuation">,</span> RSAT<span class="token operator">-</span>ADCS<span class="token punctuation">,</span> RSAT<span class="token operator">-</span>ADCS<span class="token operator">-</span>Mgmt<span class="token punctuation">,</span> RSAT<span class="token operator">-</span>NPAS<span class="token punctuation">,</span> GPMC<span class="token punctuation">,</span> Telnet<span class="token operator">-</span>Client<span class="token punctuation">,</span> RSAT<span class="token operator">-</span>ADLDS <span class="token operator">-</span>Verbose
</code></pre>
<p>The final step is to join the Management Server to the domain by executing the following PowerShell command. Enter the LAB\Administrator credentials when prompted. The server will automatically restart.</p>
<pre class=" language-powershell"><code class="prism  language-powershell"><span class="token function">Add-Computer</span> <span class="token operator">-</span>DomainName <span class="token string">"lab.local"</span> <span class="token operator">-</span>Restart
</code></pre>
<h3 id="active-directory-structure">Active Directory Structure</h3>
<p>Now that we have our Management Server up and running, lets create a structure in Active Directory for our Servers, Clients and Users. Create a structure manually by opening Active Directory Computers and Users (dsa.msc) from Server Manager or from Administrative Tools in the Control Panel or Start Menu. The script below can be used to create a simple Structure.</p>
<pre class=" language-powershell"><code class="prism  language-powershell">New<span class="token operator">-</span>ADOrganizationalUnit <span class="token operator">-</span>Name <span class="token string">"LAB"</span> <span class="token operator">-</span>Path <span class="token string">"DC=LAB,DC=local"</span>
New<span class="token operator">-</span>ADOrganizationalUnit <span class="token operator">-</span>Name <span class="token string">"Servers"</span> <span class="token operator">-</span>Path <span class="token string">"OU=LAB,DC=LAB,DC=local"</span>
New<span class="token operator">-</span>ADOrganizationalUnit <span class="token operator">-</span>Name <span class="token string">"Clients"</span> <span class="token operator">-</span>Path <span class="token string">"OU=LAB,DC=LAB,DC=local"</span>
New<span class="token operator">-</span>ADOrganizationalUnit <span class="token operator">-</span>Name <span class="token string">"Resources"</span> <span class="token operator">-</span>Path <span class="token string">"OU=LAB,DC=LAB,DC=local"</span>
New<span class="token operator">-</span>ADOrganizationalUnit <span class="token operator">-</span>Name <span class="token string">"Users"</span> <span class="token operator">-</span>Path <span class="token string">"OU=Resources,OU=LAB,DC=LAB,DC=local"</span>
New<span class="token operator">-</span>ADOrganizationalUnit <span class="token operator">-</span>Name <span class="token string">"Admins"</span> <span class="token operator">-</span>Path <span class="token string">"OU=Resources,OU=LAB,DC=LAB,DC=local"</span>
New<span class="token operator">-</span>ADOrganizationalUnit <span class="token operator">-</span>Name <span class="token string">"Service Users"</span> <span class="token operator">-</span>Path <span class="token string">"OU=Resources,OU=LAB,DC=LAB,DC=local"</span>
</code></pre>
<p>Use the following script to remove the Active Directory Example Structure and any sub OU’s within it.</p>
<pre class=" language-powershell"><code class="prism  language-powershell"><span class="token variable">$OUs</span> = Get<span class="token operator">-</span>ADObject <span class="token operator">-</span><span class="token keyword">Filter</span> <span class="token operator">*</span> <span class="token operator">-</span>SearchBase <span class="token string">'OU=LAB,DC=LAB,DC=local'</span> 
<span class="token keyword">Foreach</span> <span class="token punctuation">(</span><span class="token variable">$OU</span> in <span class="token variable">$OUs</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token function">Set</span><span class="token operator">-</span>ADObject <span class="token operator">-</span>ProtectedFromAccidentalDeletion <span class="token boolean">$false</span> <span class="token operator">-</span>Identity <span class="token variable">$OU</span>
<span class="token punctuation">}</span>
Remove<span class="token operator">-</span>ADOrganizationalUnit <span class="token operator">-</span>Identity <span class="token string">"OU=LAB,DC=LAB,DC=local"</span> <span class="token operator">-</span>Recursive <span class="token operator">-</span>Confirm:<span class="token boolean">$false</span>
</code></pre>
<p>In  <a href="../Creating-A-Configuration-Manager-(SCCM)-LAB-Environment-Part-3-Installing-SQL-Server/Installing-SQL-Server.md">part 3 we will be installing SQL Server</a>  to host the Configuration Manager database.</p>

