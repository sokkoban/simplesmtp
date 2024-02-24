# simplesmtp
<h2>Install and configure SMTP server with postfix, dovecot, fail2ban</h2>

<h4>Email server setup script</h4>
<h2>Installation</h2>
<p>https://raw.githubusercontent.com/sokkoban/simplesmtp/main/simplesmtp.sh</p>
<h2 style="text-align:start">This script installs</h2>

<ul>
	<li><strong>Postfix</strong>&nbsp;to send and receive mail.</li>
	<li><strong>Dovecot</strong>&nbsp;to get mail to your email client (mutt, Thunderbird, etc.).</li>
	<li>Config files that link the two above securely with native PAM log-ins.</li>
	<li><strong>Spamassassin</strong>&nbsp;to prevent spam and allow you to make custom filters.</li>
	<li><strong>OpenDKIM</strong>&nbsp;to validate you so you can send to Gmail and other big sites.</li>
	<li><strong>Certbot</strong>&nbsp;SSL certificates, if not already present.</li>
	<li><strong>fail2ban</strong>&nbsp;to increase server security, with enabled modules for the above programs.</li>
</ul>

<h2 style="text-align:start">This script does&nbsp;<em>not</em>...</h2>

<ul>
	<li>use a SQL database or anything like that. We keep it simple and use normal Unix system users for accounts and passwords.</li>
	<li>set up a graphical web interface for mail like Roundcube or Squirrel Mail. You are expected to use a normal mail client like Thunderbird or K-9 for Android or good old mutt with&nbsp;<a href="https://github.com/lukesmithxyz/mutt-wizard" style="box-sizing: border-box; background-color: transparent; color: var(--fgColor-accent, var(--color-accent-fg)); text-decoration: underline; text-underline-offset: 0.2rem;">mutt-wizard</a>. Note that there is a guide for&nbsp;<a href="https://landchad.net/rainloop/" rel="nofollow" style="box-sizing: border-box; background-color: transparent; color: var(--fgColor-accent, var(--color-accent-fg)); text-decoration: underline; text-underline-offset: 0.2rem;">Rainloop</a>&nbsp;on&nbsp;<a href="https://landchad.net/" rel="nofollow" style="box-sizing: border-box; background-color: transparent; color: var(--fgColor-accent, var(--color-accent-fg)); text-decoration: underline; text-underline-offset: 0.2rem;">LandChad.net</a>&nbsp;for those that want such a web interface.</li>
</ul>

<h2 style="text-align:start">Prerequisites for Installation</h2>

<ol>
	<li>Debian or Ubuntu server. I suited this script for <a href="https://www.vultr.com/?ref=8940911-8H" rel="nofollow" style="box-sizing: border-box; background-color: transparent; color: var(--fgColor-accent, var(--color-accent-fg)); text-decoration: underline; text-underline-offset: 0.2rem;">Linode</a> servers originally, but it seems to work on most other default setups on different VPS providers.</li>
	<li>DNS records that point at least your domain&#39;s&nbsp;<code>mail.</code>&nbsp;subdomain to your server&#39;s IP (IPv4 and IPv6). This is required on initial run for certbot to get an SSL certificate for your&nbsp;<code>mail.</code>&nbsp;subdomain.</li>
</ol>

<h2 style="text-align:start">Mandatory Finishing Touches</h2>

<h3 style="text-align:start">Unblock your ports</h3>

<p style="text-align:start">While the script enables your mail ports on your server, it is common practice for all VPS providers to block mail ports on their end by default. Open a help ticket with your VPS provider asking them to open your mail ports and they will do it in short order.</p>

<h3 style="text-align:start">DNS records</h3>

<p style="text-align:start">At the end of the script, you will be given some DNS records to add to your DNS server/registrar&#39;s website. These are mostly for authenticating your emails as non-spam. The 4 records are:</p>

<ol>
	<li>An MX record directing to&nbsp;<code>mail.yourdomain.tld</code>.</li>
	<li>A TXT record for SPF (to reduce mail spoofing).</li>
	<li>A TXT record for DMARC policies.</li>
	<li>A TXT record with your public DKIM key. This record is long and&nbsp;<strong>uniquely generated</strong>&nbsp;while running&nbsp;<code>emailwiz.sh</code>&nbsp;and thus must be added after installation.</li>
</ol>

<p style="text-align:start">They will look something like this:</p>

<pre>
<code>@	MX	10	mail.example.org
mail._domainkey.example.org    TXT     v=DKIM1; k=rsa; p=anextremelylongsequenceoflettersandnumbersgeneratedbyopendkim
_dmarc.example.org     TXT     v=DMARC1; p=reject; rua=mailto:dmarc@example.org; fo=1
example.org    TXT     v=spf1 mx a: -all
</code></pre>

<p style="text-align:start">The script will create a file,&nbsp;<code>~/dns_emailwiz</code>&nbsp;that will list our the records for your convenience, and also prints them at the end of the script.</p>

<h3 style="text-align:start">Add a rDNS/PTR record as well!</h3>

<p style="text-align:start">Set a reverse DNS or PTR record to avoid getting spammed. You can do this at your VPS provider, and should set it to&nbsp;<code>mail.yourdomain.tld</code>. Note that you should set this for both IPv4 and IPv6.</p>

<h2 style="text-align:start">Making new users/mail accounts</h2>

<p style="text-align:start">Let&#39;s say we want to add a user Billy and let him receive mail, run this:</p>

<pre>
<code>useradd -m -G mail info
passwd info
</code></pre>

<p style="text-align:start">Any user added to the&nbsp;<code>mail</code>&nbsp;group will be able to receive mail. Suppose a user Cassie already exists and we want to let her receive mail too. Just run:</p>

<pre>
<code>usermod -a -G mail tibo
</code></pre>

<p style="text-align:start">A user&#39;s mail will appear in&nbsp;<code>~/Mail/</code>. If you want to see your mail while ssh&#39;d in the server, you could just install mutt, add&nbsp;<code>set spoolfile=&quot;+Inbox&quot;</code>&nbsp;to your&nbsp;<code>~/.muttrc</code>&nbsp;and use mutt to view and reply to mail. You&#39;ll probably want to log in remotely though:</p>

<h2 style="text-align:start">Logging in from email clients (Thunderbird/mutt/etc)</h2>

<p style="text-align:start">Let&#39;s say you want to access your mail with Thunderbird or mutt or another email program. For my domain, the server information will be as follows:</p>

<ul>
	<li>SMTP server:&nbsp;<code>mail.domain.xyz</code></li>
	<li>SMTP port: 465</li>
	<li>IMAP server:&nbsp;<code>mail.domain.xyz</code></li>
	<li>IMAP port: 993</li>
</ul>

