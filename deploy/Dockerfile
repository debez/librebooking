# Hypothesis: apache2 binary has an MPM statically compiled in, AND
# mods-enabled has mpm_prefork symlink → duplicate MPM loaded.
# Test: remove ALL mods-enabled MPM symlinks, let Apache use whatever's
# compiled in. Dump apache2 -V to confirm.

FROM librebooking/librebooking:4.3.0

USER root

RUN echo "=== Before: mods-enabled MPM symlinks ===" \
 && (ls /etc/apache2/mods-enabled/ | grep mpm || echo "(none)") \
 && echo "=== Removing all MPM symlinks from mods-enabled ===" \
 && rm -f /etc/apache2/mods-enabled/mpm_*.load /etc/apache2/mods-enabled/mpm_*.conf \
 && echo "=== After: mods-enabled MPM symlinks ===" \
 && (ls /etc/apache2/mods-enabled/ | grep mpm || echo "(none)") \
 && echo "=== apache2 -V (shows compiled-in MPM) ===" \
 && apache2 -V \
 && echo "=== apache2ctl -t ===" \
 && (apache2ctl -t 2>&1 || true) \
 && echo "=== apache2ctl -M ===" \
 && (apache2ctl -M 2>&1 | grep -i mpm || true)
