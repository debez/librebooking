# Sledgehammer fix for upstream MPM bug.
# Disables mpm_event.so and mpm_worker.so by renaming the binaries, so even
# if something at runtime tries to LoadModule them, the .so files aren't
# present and Apache will skip them. Only mpm_prefork remains loadable.
#
# Also dumps diagnostic info so we can see what Apache modules are present.

FROM librebooking/librebooking:4.3.0

USER root

RUN echo "=== Apache MPM .so binaries before disable ===" \
 && (ls /usr/lib/apache2/modules/ | grep -i mpm || echo "(none)") \
 && echo "=== Disabling mpm_event.so and mpm_worker.so ===" \
 && (mv /usr/lib/apache2/modules/mod_mpm_event.so /usr/lib/apache2/modules/mod_mpm_event.so.disabled 2>&1 && echo "mpm_event.so renamed") \
 && (mv /usr/lib/apache2/modules/mod_mpm_worker.so /usr/lib/apache2/modules/mod_mpm_worker.so.disabled 2>&1 && echo "mpm_worker.so renamed") \
 && echo "=== Apache MPM .so binaries after disable ===" \
 && (ls /usr/lib/apache2/modules/ | grep -i mpm || echo "(none)") \
 && echo "=== mods-enabled MPM symlinks ===" \
 && (ls -la /etc/apache2/mods-enabled/ | grep -i mpm || echo "(none)") \
 && echo "=== apache2ctl -t at build time ===" \
 && (apache2ctl -t 2>&1 || true)
