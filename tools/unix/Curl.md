# ������� Curl ��� �������� �������� �������� GET, PUT, POST, DELETE

<i>Curl ������ �������, ����� ���� ����������� ������, ���������� �� ������ REST.</i>

������ *url* ���������� ������ ��� �����, ��������: *"http://dev-lab.info/api/articles"*.

## GET

<pre>curl "uri?key1=value1&amp;key2=value2"</pre>

## POST

<pre>curl -d "key1=value1&amp;key2=value2" "uri"</pre>

## PUT

� ������ ������, �������� 2 ��������.

**���� ��������� ����������� PUT ������.** ������� ���� *filename*, ��������� � ��� ������ � �������:
<pre>key1=value1&amp;key2=value2</pre>
����� ��������� ������:
<pre>curl -T filename "uri"</pre>

**���� ��������� ������ ������� PUT, �� �����, ��� ������������, ��� �������� ���
�� ������� GET - �� ����� � ����������**. � ���� ������, ���� ������� ����, �� ��������� ��� ��������� ������.
� ����� ��������� ��� ��������� ���� �������.

## DELETE

<pre>curl -X DELETE "uri"</pre>

������:
<pre>curl -u natalie:mypass -c cookie.txt -b cookie.txt -X DELETE http://dev-lab.info/api/articles/45</pre>
�����:
<pre>HTTP/1.1 200 OK
Server: nginx/1.0.3
Date: Thu, 27 Dec 2012 10:00:08 GMT
Content-Type: application/json
Connection: keep-alive
Vary: Content-Type
Content-Length: 2
Set-Cookie: sid=99c14ab8495958586fa06ae60d5ecaaaef13f23c; path=/; expires=Sun, 27-Dec
-2012 10:00:08 GMT; HttpOnly
Status: 200

{}
</pre>
