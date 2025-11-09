# Доработки netlab

_netlab_ хорошо [поддается](https://netlab.tools/dev/guidelines/) доработкам в любом виде. В отчетах о лабораторных отмечено где в ходе выполнения дописывались дополнительные параметры или реализовывалась незадокументированная поддержка.&#x20;

При наличии свободного времени надеюсь внести pull request, если вы это читаете и обладаете свободным временем и необходимыми знаниями можете сделать это за меня.



1\) включить демона bfd для frr по умолчанию(прописывается в файле /usr/local/lib/python3.10/dist-packages/netsim/templates/provider/clab/frr/daemons.j2), или что еще лучше включить по умолчанию всех, т.к. для работы с любым протоколом frr должен быть запущен с включенным демоном протокола.

2\)  добавление поддержки bfd для isis (/usr/local/lib/python3.10/dist-packages/netsim/ansible/templates/isis/frr.macro.j2) для frr

3\) добавление поддержки bfd для ipv6 /usr/local/lib/python3.10/dist-packages/netsim/ansible/templates/isis/eos.macro.j2 для eos(arista)

4\) добавить параметр rr\_mesh в /usr/local/lib/python3.10/dist-packages/netsim/modules/bgp.yml который будет управлять созданием сессии между bgp route-reflector , по умолчанию выставить true.



