# Workaround for upstream librebooking/librebooking image bug:
# 4.3.0 ships with BOTH mpm_event AND mpm_prefork symlinked in
# /etc/apache2/mods-enabled/, which makes Apache refuse to start with:
#   AH00534: Configuration error: More than one MPM loaded.
#
# Verified via runtime diagnostic on 2026-05-06:
#   /etc/apache2/mods-enabled/mpm_event.{conf,load}     ← shouldn't exist
#   /etc/apache2/mods-enabled/mpm_prefork.{conf,load}   ← keep (mod_php needs prefork)
#
# Fix: remove mpm_event and mpm_worker symlinks. mpm_prefork stays.
# Track upstream and delete this Dockerfile when fixed.

FROM librebooking/librebooking:4.3.0

USER root

RUN echo "=== Before: mods-enabled MPM symlinks ===" \
 && ls /etc/apache2/mods-enabled/ | grep mpm \
 && rm -fv /etc/apache2/mods-enabled/mpm_event.* /etc/apache2/mods-enabled/mpm_worker.* \
 && echo "=== After: mods-enabled MPM symlinks ===" \
 && ls /etc/apache2/mods-enabled/ | grep mpm
