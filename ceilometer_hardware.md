# Ceilometer hardware 目录代码



snmp.py
class SNMPInspector(base.Inspector):
...

392         def inspect_generic(self, host, identifier, cache, extra_metadata=None):
393             import pdb
394             pdb.set_trace()
395             # the snmp definition for the corresponding meter
396  ->         meter_def = self.MAPPING[identifier]
397             # collect oids that needs to be queried
398             oids_to_query = self._find_missing_oids(meter_def, cache)
399             # query oids and populate into caches
400             if oids_to_query:
401                 self._query_oids(host, oids_to_query, cache,
(Pdb) p host
SplitResult(scheme=u'snmp', netloc=u'192.168.100.9', path=u'', query='', fragment='')
(Pdb) p identifier
'cpu.load.1min'
(Pdb) p cache
{}
(Pdb) p extra_metadata
{'resource_url': u'snmp://192.168.100.9', 'resource_id': u'controller'}
