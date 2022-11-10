# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define :authz do |authz|
    authz.hostname = "SAH-authz"
    authz.vm.box = "ubuntu/trusty64"
    authz.vm.network :private_network, ip: "10.0.2.15"
    authz.vm.provision "shell", run: 'once', inline: <<-SHELL
      # apt-get update
      export SLAPD_D='/etc/ldap/slapd.d'
      mkdir -p /etc/ldap/
      touch /etc/ldap/noslapd
      export DEBIAN_FRONTEND=noninteractive
      apt-get install -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" ldap-utils slapd || true
      mkdir -p $SLAPD_D
      rm -rf $SLAPD_D/*
      install -d -C -o openldap -g openldap -m 0700 /var/lib/ldap/sah
      /usr/sbin/slapadd -f /vagrant/slapd.conf -l /vagrant/init.ldif
      chown -Rc openldap:openldap /var/lib/ldap/sah
      /usr/sbin/slaptest -f /vagrant/slapd.conf -F $SLAPD_D
      chown -Rc openldap:openldap $SLAPD_D
      rm /etc/ldap/noslapd
      service slapd start
      install -C -o root -g root -m 0600 /vagrant/ldap.conf /etc/ldap/
      ldapadd -f /vagrant/openssh-lpk.schema.ldif
      ldapadd -f /vagrant/populate.ldif

      cat /vagrant/ldap-auth-config.debconf | debconf-set-selections

      apt-get install -y libnss-ldap
       /usr/bin/install -C -o root -g root -m 0644 /vagrant/nsswitch.conf /etc
       /usr/bin/install -C -o root -g root -m 0640 /vagrant/SAH.KRL /etc
       /usr/bin/install -C -o root -g root -m 0555 /vagrant/get-ssh-auth-keys-from-ldap /etc/ssh
       /usr/bin/install -C -o root -g root -m 0644 /vagrant/sshd_config /etc/ssh/

       service ssh restart
    SHELL
  end

  config.vm.define :sahserver do |sahserver|
    sahserver.vm.box = "ubuntu/trusty64"
    sahserver.vm.hostname = "sahserver"
    sahserver.vm.network :private_network, ip: "10.0.2.16"

    sahserver.vm.provision "shell", run: 'once', inline: <<-SHELL
      debconf-get-selections /vagrant/ldap-auth-config.debconf

      apt-get install -y libnss-ldap
      /usr/bin/install -C -o root -g root -m 0644 /vagrant/nsswitch.conf /etc
      /usr/bin/install -C -o root -g root -m 0640 /vagrant/SAH.KRL /etc
      /usr/bin/install -C -o root -g root -m 0555 /vagrant/get-ssh-auth-keys-from-ldap /etc/ssh
      /usr/bin/install -C -o root -g root -m 0644 /vagrant/sshd_config /etc/ssh/
      service ssh restart
    SHELL
  end
end
