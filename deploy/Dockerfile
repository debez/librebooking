# Workaround for upstream librebooking/librebooking image bug:
# 4.3.0 ships with BOTH mpm_event AND mpm_prefork symlinked in
# /etc/apache2/mods-enabled/, which makes Apache refuse to start with:
#   AH00534: Configuration error: More than one MPM loaded.
#
# Verified via runtime diagnostic on 2026-05-06 — both mpm_event and
# mpm_prefork are present in mods-enabled at runtime.
#
# Fix: nuke the mpm_event and mpm_worker symlinks so only mpm_prefork
# remains. mod_php requires mpm_prefork (not thread-safe).
#
# Cache-bust: 2026-05-06b
# Track upstream and delete this Dockerfile when fixed.

FROM librebooking/librebooking:4.3.0

USER root

RUN set -eux \
 && echo "Before:" && ls -la /etc/apache2/mods-enabled/ | grep -i mpm \
 && rm -v /etc/apache2/mods-enabled/mpm_event.conf \
 && rm -v /etc/apache2/mods-enabled/mpm_event.load \
 && rm -fv /etc/apache2/mods-enabled/mpm_worker.conf \
 && rm -fv /etc/apache2/mods-enabled/mpm_worker.load \
 && echo "After:" && ls -la /etc/apache2/mods-enabled/ | grep -i mpm \
 && echo "Loaded MPMs (build-time):" && apache2ctl -M 2>/dev/null | grep -i mpm
