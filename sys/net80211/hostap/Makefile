# $OpenBSD$

#
# Simple plain WLAN ping test
#
# This tests configures a WLAN access point and a client host.
# The client connects to the AP and tries to ping it.

# These options are used to configure the AP and the client for this test.
AP_IF ?=	rum0
HT_IF ?=	iwm0
AP_IP ?=	10.190.0.1
HT_IP ?=	10.190.0.2
AP_RD ?=	1
HT_RD ?=	2
AP_NWID ?=	regress
AP_BSSID !=	ifconfig rum0 | sed -ne 's/.*lladdr \(..:..:..:..:..:..\)$$/\1/p'

#d8:5d:4c:8f:7e:19

.for node in AP HT
unconfig-${node}:
	ifconfig ${${node}_IF} -bssid -chan -nwid -nwkey -wpa -wpakey \
	    -inet -inet6 down
.endfor

.for auth in plain
config-AP-${auth}: unconfig-AP
	ifconfig ${AP_IF} mediaopt hostap nwid ${AP_NWID}
	ifconfig ${AP_IF} inet ${AP_IP}/24 rdomain ${AP_RD}

config-HT-${auth}: unconfig-HT
	ifconfig ${HT_IF} nwid ${AP_NWID} bssid ${AP_BSSID} 
	ifconfig ${HT_IF} inet ${HT_IP}/24 rdomain ${HT_RD}
.endfor

.for auth in plain

stamp-${auth}-config:
	${.MAKE} -C ${.CURDIR} config-AP-${auth}
	${.MAKE} -C ${.CURDIR} config-HT-${auth}
	until ifconfig ${AP_IF} | grep -q 'status: active'; do sleep 0.1; done
	until ifconfig ${HT_IF} | grep -q 'status: active'; do sleep 0.1; done
	touch $@

.for node src_rd dst_rd in AP ${HT_RD} ${AP_RD} HT ${AP_RD} ${HT_RD}

REGRESS_TARGETS += 80211-${auth}-ping-${node}

80211-${auth}-ping-${node}: stamp-${auth}-config
	ping -n -c 1 -V ${src_rd} ${${node}_IP}

REGRESS_TARGETS += 80211-${auth}-bench-${node}

80211-${auth}-bench-${node}: stamp-${auth}-config
	sleep 1; tcpbench -V ${dst_rd} -s &
	@echo "=== start of $@ data ==="
	sleep 2; tcpbench -V ${src_rd} -t 30 ${${node}_IP}
	@echo "=== end of $@ data ==="
	pkill -x tcpbench
.endfor

REGRESS_TARGETS += 80211-${auth}-cleanup

80211-${auth}-cleanup:
	${.MAKE} -C ${.CURDIR} unconfig-AP
	${.MAKE} -C ${.CURDIR} unconfig-HT
	rm -f stamp-${auth}-config

.endfor

.include <bsd.regress.mk>
