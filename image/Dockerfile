# Use osixia/light-baseimage
# sources: https://github.com/osixia/docker-light-baseimage
FROM osixia/light-baseimage:1.3.2

ARG OPENLDAP_PACKAGE_VERSION=2.4.57

ARG LDAP_OPENLDAP_GID
ARG LDAP_OPENLDAP_UID

ARG PQCHECKER_VERSION=2.0.0
ARG PQCHECKER_MD5=c005ce596e97d13e39485e711dcbc7e1

# Add openldap user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
# If explicit uid or gid is given, use it.
RUN if [ -z "${LDAP_OPENLDAP_GID}" ]; then groupadd -g 911 -r openldap; else groupadd -r -g ${LDAP_OPENLDAP_GID} openldap; fi \
    && if [ -z "${LDAP_OPENLDAP_UID}" ]; then useradd -u 911 -r -g openldap openldap; else useradd -r -g openldap -u ${LDAP_OPENLDAP_UID} openldap; fi

# Switch to archive repositories since Buster is EOL (end-of-life)
# and add buster-backports for newer openldap components
#
# Steps:
# 1. Replace deb.debian.org with archive.debian.org (main repo moved to archive)
# 2. Replace security.debian.org with archive.debian.org (security repo moved to archive)
# 3. Remove buster-updates line (doesn't exist in archive)
# 4. Add buster-backports from archive for newer OpenLDAP packages
# 5. Disable timestamp validation (archived repos have expired timestamps)
# 6. Temporarily allow insecure repos (needed to install gnupg)
# 7. Install gnupg and dirmngr (required for importing GPG keys)
# 8. Import missing Debian archive signing keys from Ubuntu keyserver
# 9. Remove insecure repos config (security restored after key import)
# 10. Update package lists with verified signatures
# 11. Remove gnupg/dirmngr (no longer needed, reduces image size)
# 12. Clean up apt cache
RUN ARCHIVE_URL="archive.debian.org" && \
    sed -i "s|http://deb.debian.org/debian|https://${ARCHIVE_URL}/debian|g" /etc/apt/sources.list && \
    sed -i "s|http://security.debian.org/debian-security|https://${ARCHIVE_URL}/debian-security|g" /etc/apt/sources.list && \
    sed -i "s|https://deb.debian.org/debian|https://${ARCHIVE_URL}/debian|g" /etc/apt/sources.list && \
    sed -i "s|https://security.debian.org/debian-security|https://${ARCHIVE_URL}/debian-security|g" /etc/apt/sources.list && \
    grep -q "buster-updates" /etc/apt/sources.list && sed -i '/buster-updates/d' /etc/apt/sources.list || true && \
    echo "deb https://${ARCHIVE_URL}/debian buster-backports main" >> /etc/apt/sources.list && \
    echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid-until && \
    echo 'Acquire::AllowInsecureRepositories "true";' > /etc/apt/apt.conf.d/99allow-insecure && \
    apt-get update && \
    apt-get install -y --no-install-recommends gnupg dirmngr && \
    apt-key adv --keyserver hkps://keyserver.ubuntu.com --recv-keys 0E98404D386FA1D9 6ED0E7B82643E131 && \
    rm /etc/apt/apt.conf.d/99allow-insecure && \
    apt-get update && \
    apt-get remove -y --purge --auto-remove gnupg dirmngr && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install OpenLDAP, ldap-utils and ssl-tools from the (backported) baseimage and clean apt-get files
# sources: https://github.com/osixia/docker-light-baseimage/blob/master/image/tool/add-service-available
#          https://github.com/osixia/docker-light-baseimage/blob/master/image/service-available/:ssl-tools/download.sh
RUN echo "path-include /usr/share/doc/krb5*" >> /etc/dpkg/dpkg.cfg.d/docker && apt-get -y update \
    && /container/tool/add-service-available :ssl-tools \
    && LC_ALL=C DEBIAN_FRONTEND=noninteractive apt-get -t buster-backports install -y --no-install-recommends \
    ca-certificates \
    curl \
    ldap-utils=${OPENLDAP_PACKAGE_VERSION}\* \
    libsasl2-modules \
    libsasl2-modules-db \
    libsasl2-modules-gssapi-mit \
    libsasl2-modules-ldap \
    libsasl2-modules-otp \
    libsasl2-modules-sql \
    openssl \
    slapd=${OPENLDAP_PACKAGE_VERSION}\* \
    slapd-contrib=${OPENLDAP_PACKAGE_VERSION}\* \
    krb5-kdc-ldap \
    && update-ca-certificates \
    && apt-get remove -y --purge --auto-remove curl ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Add service directory to /container/service
ADD service /container/service

# Use baseimage install-service script
# https://github.com/osixia/docker-light-baseimage/blob/master/image/tool/install-service
RUN /container/tool/install-service

# Add default env variables
ADD environment /container/environment/99-default

# Expose default ldap and ldaps ports
EXPOSE 389 636

# Put ldap config and database dir in a volume to persist data.
# VOLUME /etc/ldap/slapd.d /var/lib/ldap
