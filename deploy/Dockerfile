# Minimal pass-through. Need root for the runtime mpm_event cleanup
# (which is set as the service's Custom Start Command on Railway since
# build-time changes to /etc/apache2/mods-enabled aren't sticking through
# to runtime).

FROM librebooking/librebooking:4.3.0
USER root
