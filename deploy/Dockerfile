# DIAGNOSTIC BUILD — does not modify the image, just dumps Apache state
# so we can figure out why "More than one MPM loaded" happens at runtime
# despite only mpm_prefork being symlinked in mods-enabled at build time.

FROM librebooking/librebooking:4.3.0

USER root

RUN echo "=== id ===" \
 && id \
 && echo "=== APACHE_* env vars ===" \
 && (env | grep -i apache || echo "(none)") \
 && echo "=== apache2 -V ===" \
 && (apache2 -V 2>&1 || true) \
 && echo "=== mods-enabled (full) ===" \
 && ls -la /etc/apache2/mods-enabled/ \
 && echo "=== conf-enabled ===" \
 && ls -la /etc/apache2/conf-enabled/ \
 && echo "=== sites-enabled ===" \
 && ls -la /etc/apache2/sites-enabled/ \
 && echo "=== Full LoadModule grep across /etc/apache2 ===" \
 && (grep -rn -i 'loadmodule' /etc/apache2/ 2>/dev/null || echo "(none)") \
 && echo "=== apache2ctl -t (config validation) ===" \
 && (apache2ctl -t 2>&1 || true) \
 && echo "=== apache2ctl -M (loaded modules) ===" \
 && (apache2ctl -M 2>&1 || true) \
 && echo "=== END DIAGNOSTIC ==="
