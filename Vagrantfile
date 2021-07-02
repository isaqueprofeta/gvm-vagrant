GVMD_USER = "admin"
GVMD_PASS = "Vu1nCh3ck3R"

Vagrant.configure("2") do |config|

	config.vm.define "gvm" do |gvm|

		config.vm.box = "generic/alpine313"

		gvm.vm.hostname = "gvm"
		gvm.vm.network "forwarded_port", guest: 9392, host: 9392, host_ip: "127.0.0.1"
        gvm.vm.network "forwarded_port", guest: 5432, host: 5432, host_ip: "127.0.0.1"

		gvm.vm.provision "shell", inline: <<-SHELL
			echo '----Installing Packages---- '
			apk add openvas openvas-config gvmd gvm-libs greenbone-security-assistant ospd-openvas
			SHELL

		gvm.vm.provision "shell", inline: <<-SHELL
			echo '----Configuring Postgres---- '
			rc-service postgresql setup
            rc-service postgresql start
            rc-update add postgresql
			SHELL

		gvm.vm.provision "shell", inline: <<-SHELL
            echo '----Creating database, user and extensions---- '
            sudo -u postgres psql -c "CREATE USER gvm WITH ENCRYPTED PASSWORD 'Vu1nCh3ck3R'"
            sudo -u postgres createdb -O gvm gvmd
            echo 'CREATE ROLE dba WITH SUPERUSER NOINHERIT;' | sudo -u postgres psql gvmd
            echo 'GRANT dba TO gvm;' | sudo -u postgres psql gvmd
            echo 'CREATE EXTENSION IF NOT EXISTS \"uuid-ossp\";' | sudo -u postgres psql gvmd
            echo 'CREATE EXTENSION IF NOT EXISTS \"pgcrypto\";' | sudo -u postgres psql gvmd
			SHELL

        gvm.vm.provision "shell", inline: <<-SHELL
            echo '----Create certificate and start gvmd---- '
            sudo -u gvm gvm-manage-certs -a
            rc-service gvmd start
            rc-update add gvmd
            SHELL

        gvm.vm.provision "shell", inline: <<-SHELL
            echo '----Create credentials used to interact with gvmd---- '
            sleep 5
            sudo -u gvm gvmd --create-user=#{GVMD_USER} --password=#{GVMD_PASS}
            sudo -u gvm gvmd --modify-setting 78eceaec-3385-11ea-b237-28d24461215b --value $(sudo -u gvm gvmd --get-users --verbose | cut -f2 -d" ")
			SHELL

        gvm.vm.provision "shell", inline: <<-SHELL
            echo '----Download the GVM and NVT definitions---- '
            sudo -u gvm greenbone-feed-sync --type GVMD_DATA
            sudo -u gvm greenbone-feed-sync --type SCAP
            sudo -u gvm greenbone-feed-sync --type CERT
            sudo -u gvm greenbone-nvt-sync
            rm -rf /run/ospd/feed-update.lock
            sudo -u gvm greenbone-nvt-sync
			SHELL


        gvm.vm.provision "shell", inline: <<-SHELL
            echo '----Setting up the scheduller for GVM and NVT definitions---- '
            echo "0 3 * * * sudo -u gvm greenbone-feed-sync --type GVMD_DATA" | crontab -
            (crontab -l && echo "0 4 * * * sudo -u gvm greenbone-feed-sync --type SCAP") | crontab -
            (crontab -l && echo "0 5 * * * sudo -u gvm greenbone-feed-sync --type CERT") | crontab -
            (crontab -l && echo "0 6 * * * sudo -u gvm greenbone-nvt-sync") | crontab -
			SHELL

        gvm.vm.provision "shell", inline: <<-SHELL
            echo '----Configure Greenbone Security Assistant---- '
            echo 'GSAD_LISTEN_ADDRESS="0.0.0.0"' > /etc/conf.d/gsad
            rc-service gsad start
            rc-update add gsad
			SHELL

    end

end
