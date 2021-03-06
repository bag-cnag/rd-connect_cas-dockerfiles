FROM	rd-connect.eu/rdconnect-umi-prereqs:latest
MAINTAINER	Andrés Cañada Pallarés <acanada@cnio.es>, José María Fernández <jmfernandez@cnio.es>
LABEL	Description="RD-Connect User Management Interface" Vendor="CNIO"

#UMI_TAG=latest
#mkdir -p "${PWD}"/umi_rd-connect/tmp


ARG	USERMANAGEMENT_BRANCH=ldap
ARG	USERMANAGEMENT_CHECKOUT=9ba94da2ef3d64b44cac5225b2b2beaae48a09c6
ARG	UMI_BRANCH=master
ARG	UMI_CHECKOUT=442241a8bd3f37332b3de31a0629fd5e826c4fc6

ARG	CAS_SERVER_URL=https://rdconnectcas.rd-connect.eu:9443/cas
ARG	LDAP_SERVER=ldap.rd-connect.eu
ARG	CAS_LDAP_USER="cn=admin,dc=rd-connect,dc=eu"
# This is a placeholder for the true intermediate LDAP admin password
ARG	CAS_LDAP_PASS=changeit

RUN	useradd -m -U -c 'RD-Connect REST API unprivileged user' rdconnect-rest
# Configure apache for application
# User was created in the prerequisites dockerfile
USER	rdconnect-rest
ENV	HOME=/home/rdconnect-rest
WORKDIR	${HOME}

RUN	git clone -b "${USERMANAGEMENT_BRANCH}" https://github.com/inab/rd-connect-user-management.git && \
	cd rd-connect-user-management && \
	git checkout "$USERMANAGEMENT_CHECKOUT"
RUN	cd rd-connect-user-management && ./install-deps.sh
RUN	mkdir -p RDConnect-UserManagement-REST-API
RUN	cd rd-connect-user-management && cp -dpr configs user-management.cgi user-management.fcgi user-management.psgi libs "$HOME"/RDConnect-UserManagement-REST-API && cp template-config.ini "$HOME"/RDConnect-UserManagement-REST-API/configs/user-management.ini
RUN	sed -i "s/^.*ldap_host=.*/ldap_host=${LDAP_SERVER}/; s/^.*ldap_user=.*/ldap_user=${CAS_LDAP_USER}/; s/^.*ldap_pass=.*/ldap_pass=${CAS_LDAP_PASS}/; s#^cas_url=.*#cas_url=${CAS_SERVER_URL}#;" RDConnect-UserManagement-REST-API/configs/user-management.ini

ENV	UMI_CLONE=${HOME}/user-management-interface

RUN	git clone -b "${UMI_BRANCH}" https://github.com/inab/rd-connect-user-management-interface.git "${UMI_CLONE}" && \
	cd "${UMI_CLONE}" && \
	git checkout "${UMI_CHECKOUT}"
WORKDIR ${UMI_CLONE}
RUN	npm install
RUN	PATH="${UMI_CLONE}/node_modules/.bin:$PATH" gulp build
RUN	mkdir -p "$HOME/DOCUMENT_ROOT" && cp -dpTr build "$HOME/DOCUMENT_ROOT/user-management"

# The SELinux part
#RUN	chmod go+rx "$HOME" && \
#	chcon -Rv --type=httpd_sys_content_t "$HOME"/DOCUMENT_ROOT && \
#	chcon -Rv --type=httpd_sys_content_t "$HOME"/rd-connect-user-management && \
#	chcon -Rv --type=httpd_sys_content_t "$HOME"/RDConnect-UserManagement-REST-API && \
#	chcon -Rv --type=httpd_sys_script_exec_t "$HOME"/RDConnect-UserManagement-REST-API/user-management.cgi

USER	root
ENV	HOME=/root
WORKDIR	${HOME}

RUN	yum --enablerepo=epel install -y httpd-itk
#RUN	yum install -y checkpolicy policycoreutils-python && \
#	setsebool -P httpd_enable_cgi=1 && \
#	setsebool -P httpd_read_user_content=1 && \
#	setsebool -P httpd_can_network_connect=1 && \
#	setsebool -P httpd_enable_homedirs=1 && \
#	cd /tmp && \
#	checkmodule -M -m -o rdconnect-user-management.mod /home/rdconnect-rest/rd-connect-user-management/selinux/rdconnect-user-management.te && \
#	semodule_package -o rdconnect-user-management.pp -m rdconnect-user-management.mod && \
#	semodule -i rdconnect-user-management.pp
RUN	sed -i 's/^#\(LoadModule \)/\1/' /etc/httpd/conf.modules.d/00-mpm-itk.conf
COPY	umi.conf /tmp
RUN	sed -e '/<\/VirtualHost>/r /tmp/umi.conf' -e 'x;$G' -i /etc/httpd/conf.d/ssl.conf && rm /tmp/umi.conf

# Enabling the API as such


# Cleaning up
RUN	yum autoremove -y git
RUN	yum clean all

VOLUME	[ "/home/rdconnect-rest/RDConnect-UserManagement-REST-API/configs"]
