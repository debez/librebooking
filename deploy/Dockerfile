# Thin overlay over upstream librebooking/librebooking.
#
# Workaround: upstream image rebuilds between 2026-04-30 and 2026-05-06 ship with
# both mpm_event and mpm_prefork Apache modules loaded, causing Apache to refuse
# to start with: AH00534: Configuration error: More than one MPM loaded.
# LibreBooking uses mod_php, which requires mpm_prefork — disable event/worker.
#
# Track the upstream issue and delete this Dockerfile (switch the Railway service
# back to deploying from the upstream image directly) once fixed.

FROM librebooking/librebooking:4.3.0

RUN a2dismod -f mpm_event mpm_worker 2>/dev/null || true \
 && a2enmod mpm_prefork
