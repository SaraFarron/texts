# ������ � cookies � CGI::Application

## ��������� cookies

<pre>sub on_set_cookies {
	my $self = shift;
	my $query = $self-&gt;query();

	my $auth_cookie = $query-&gt;cookie( # �������������� cookie
			-name=&gt;'alisa',
			-value=&gt;'pass',
			-expires=&gt;'+1h',
			-path=&gt;'/cgi',
			-domain=&gt;'.aninatalie.ru',
	);

	$self-&gt;header_props(-cookie=&gt;[$auth_cookie]); # ������������� � �������

	return $self-&gt;tt_process('template2.tt');
}

</pre>

## ��������� ���������� �� cookies

<pre>sub on_set_cookies {
	my $self = shift;
	my $query = $self-&gt;query();

	my $auth_cookie = $query-&gt;cookie('alisa');
		# �������� ���������� $auth_cookie ����� ����� 'pass'. 
		# �.�. �� ������� ������ �������� -value .
	...
	return $self-&gt;tt_process('template2.tt');
}
</pre>

## ���������� ������ cookies

����� �������� �������� <font color="#00aa00">cookie</font>, ���� ������ �������� ����� ������ ������ ������������. ��� ����� ��������� <font color="#00aa00">-name</font> ������ <font color="#00aa00">cookie</font>, � ������ ����� ��������� ��� ���.

## �������� cookies

������� <font color="#00aa00">cookie</font> ���� ������ - ��������� �������� ������ <font color="#00aa00">�ookie</font> � <font color="#00aa00">-expires</font> ����������� ������������� ��������, ��������, <font color="#00aa00">-1d</font>. ������� ������ ���������� <font color="#00aa00">�ookie</font> �, ������ ������������� �����, ���������, ��� �� ������ ����� ��� ������ ��� ������� <font color="#00aa00">�ookie</font> (� ������ ������, ��� �����). � ������.
<pre>sub on_set_cookies {
	my $self = shift;
	my $query = $self-&gt;query();

	my $auth_cookie = $query-&gt;cookie(
			-name=&gt;'alisa',
			-value=&gt;'pass',
			-expires=&gt;'-1h',
			-path=&gt;'/cgi',
			-domain=&gt;'.aninatalie.ru',
	);

	$self-&gt;header_props(-cookie=&gt;[$auth_cookie]);

	return $self-&gt;tt_process('template2.tt');
}
</pre>

## �������
### ��� ����� cookies

<font color="#00aa00">Cookie</font> �������� �������� ����� �� �������������� ������� HTTP ��������� (HyperText Transfer Protocol). �������� ����������� � ������������� ���������� ����� �������� � ��������.

��������� <font color="#00aa00">cookie</font>, ����� ����������� ������ �� HTTP ���������. ������� ������� �������� ������ �����: �� ������ ������� �������� �������������� �������� <font color="#00aa00">cookie</font>, � ��� ������ ����������� ������� ��� �������� �������� �� ���������� ��������� HTTP_COOKIE � ��������������� ������� ��������������.

<font color="#00aa00">Cookie</font> - ��� ��������� ������ ��������� ����������, ������� ������ �������� ��������. ������� ����� ������� ��� ���������� � ���������� �� ������� � ������ �������� ��� ����� HTTP ���������.

��������� <font color="#00aa00">cookie</font> ����� ��������� ������ � ������� ����� ������, ��� ��������� ����� �������� ��������. ������, ������������� �� ������������ ������ �������, ������������ � ����(�).

### ������ � ��������� cookie

<font color="#00aa00">�ookie �������� ������ HTTP ���������</font>. ������ �������� ���� Set-Cookie HTTP ���������:
<pre>Set-Cookie: name=VALUE; expires=DATE; path=PATH; domain=DOMAIN_NAME; secure</pre>

����������� �������� ���� Set-Cookie HTTP ���������:
<pre>Set-Cookie: name=VALUE;</pre>

<ul>
<li><b>name=VALUE</b> - ������ ��������, �������� ������� ������, ������� � �������. NAME-���
cookie, VALUE - ��������. �� ����������� ������������� ���������, ������� � �������.</li>

<li><b>expires=DATE</b> - ����� �������� cookie, �.�. ������ DATE ������ ������ ���� � �������
"expires=Monday, DD-Mon-YYYY HH:MM:SS GMT", ����� ������� �������� ����� �������� cookie. ����
���� ������� �� ������, �� cookie �������� � ������� ������ ������, �� �������� ��������.</li>

<li><b>domain=DOMAIN_NAME</b> - �����, ��� �������� �������� cookie �������������. ��������,
"domain=cit-forum.com". ���� ���� ������� ������, �� �� ��������� ������������ �������� ��� �������, �� ������� ����
������ �������� cookie.</li>

<li><b>path=PATH</b> - ������� ������������� ������������ ����������, ��� ������� �������������
�������� cookie. ��� ����, ����� cookie ����������
��� ������ ������� � �������, ���������� ������� �������� ������� �������, ��������, "path=/".
���� ���� ������� �� ������, �� �������� cookie ���������������� ������ �� ��������� � ��� ��
����������, ��� � ��������, � ������� ���� ����������� �������� cookie.</li>

<li><b>secure</b> - ���� ����� ���� ������, �� ���������� cookie ������������ ������ �����
HTTPS (HTTP � �������������� SSL - Secure Socket Level), � ���������� ������. ���� ����
������ �� ������, �� ���������� ������������ ������� ��������.</li>
</ul>
