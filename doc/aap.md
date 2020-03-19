1. Configure AAP for both the VMIs and also associate FIP to them.
2. Via VNC API, read the FIP object and set its 'floating_ip_fixed_ip_address' property to the AAP IP.

FIP will now follow the active member of the group.

>>> from vnc_api.vnc_api import *
>>> client = VncApi(api_server_host='127.0.0.1', api_server_port=8082, username='admin', password='contrail123', tenant_name='admin')
>>> fip = client.floating_ip_read(id='ceb88d00-64bc-4ba1-8639-7b312a87b767')
>>> fip.floating_ip_fixed_ip_address = '10.0.0.50'
>>> client.floating_ip_update(fip)
u'{"floating-ip": {"href": "http://127.0.0.1:8082/floating-ip/ceb88d00-64bc-4ba1-8639-7b312a87b767", "uuid": "ceb88d00-64bc-4ba1-8639-7b312a87b767"}}'


