# Thin overlay over upstream librebooking/librebooking.
#
# Workaround: upstream image rebuilds between 2026-04-30 and 2026-05-06 ship with
# multiple Apache MPM modules loaded, causing Apache to refuse to start with:
#   AH00534: Configuration error: More than one MPM loaded.
# LibreBooking uses mod_php, which requires mpm_prefork. This overlay nukes any
# MPM symlinks in mods-enabled and explicitly enables only mpm_prefork.
#
# Diagnostic output is printed during build so we can verify the fix took.
# Track the upstream issue and delete this Dockerfile (switch the Railway service
# back to deploying from the upstream image directly) once fixed.

FROM librebooking/librebooking:4.3.0

RUN echo "=== Before cleanup: mods-enabled MPM symlinks ===" \
 && (ls -la /etc/apache2/mods-enabled/ | grep -i mpm || echo "(none)") \
 && echo "=== Before cleanup: LoadModule mpm in apache2.conf ===" \
 && (grep -i 'loadmodule.*mpm' /etc/apache2/apache2.conf || echo "(none in apache2.conf)") \
 && echo "=== Before cleanup: any LoadModule mpm anywhere in /etc/apache2 ===" \
 && (grep -ril 'loadmodule.*mpm' /etc/apache2/ || echo "(none)") \
 && echo "=== Removing all MPM symlinks from mods-enabled ===" \
 && rm -fv /etc/apache2/mods-enabled/mpm_*.load /etc/apache2/mods-enabled/mpm_*.conf \
 && a2enmod mpm_prefork \
 && echo "=== After cleanup: mods-enabled MPM symlinks ===" \
 && (ls -la /etc/apache2/mods-enabled/ | grep -i mpm || echo "(none)") \
 && echo "=== apache2ctl -t (config validation) ===" \
 && apache2ctl -t
