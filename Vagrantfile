# -*- mode: ruby -*-
# vi: set ft=ruby :

rbenv_rubies = ENV['RBENV_RUBIES'] ? ENV['RBENV_RUBIES'].split(' ') : %w{1.9.3-p194}

s3_bash_url = "https://github.com/cosmin/s3-bash/tarball/master"

aws_access_key_id     = ENV['AWS_ACCESS_KEY_ID']
aws_secret_access_key = ENV['AWS_SECRET_ACCESS_KEY']
s3_bucket             = ENV['S3_BUCKET'] && ENV['S3_BUCKET'].sub(%r{^([^/])}, '/\1')

Vagrant::Config.run do |config|
  config.vm.box = ENV['VAGRANT_BOX'] || "base"

  config.vm.provision :shell do |shell|
    shell.inline = <<-PROVISION.gsub(/^ {6}/, '')
      if which apt-get >/dev/null ; then
        preseed=/var/cacle/local/preseeding/grub.seed
        mkdir -p $(dirname $preseed)
        echo 'grub-pc	grub-pc/install_devices	multiselect	/dev/sda1' > $preseed
        debconf-set-selections $preseed

        apt-get -y update
        apt-get -y upgrade
      fi
    PROVISION
  end

  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = "cookbooks"

    chef.add_recipe "java"
    chef.add_recipe "ruby_build"
    chef.add_recipe "rbenv::system"

    chef.json = { 'rbenv' => { 'rubies' => rbenv_rubies } }
  end

  config.vm.provision :shell do |shell|
    shell.inline = <<-PROVISION.gsub(/^ {6}/, '')
      set -e

      s3bp="$HOME/s3-bash"
      if [ ! -d "$s3bp" ] ; then
        mkdir -p "$s3bp"
        (cd "$s3bp" && curl -sL '#{s3_bash_url}' | tar xzf - --strip-components 1)
      fi
      export PATH="$s3bp:$PATH"

      echo 'x-amz-acl: public-read' > "$HOME/aws_s3_headers.txt"
      echo '#{aws_access_key_id}' > "$HOME/aws_access_key_id.txt"
      echo -n '#{aws_secret_access_key}' > "$HOME/aws_secret_access_key.txt"
      chmod 0600 $HOME/aws_{access_key_id,secret_access_key}.txt

      os=$(lsb_release -s -i | tr '[[:upper:]]' '[[:lower:]]')
      release=$(lsb_release -s -r)
      arch=$(uname -m)

      tarfile () {
        local version="$1";
        echo "rbenv-system-${os}-${release}-${version}-${arch}.tar"
      }

      for r in #{rbenv_rubies.join(' ')} ; do
        echo "----> Processing rbenv Ruby version $r..."
        s3_url="https://s3.amazonaws.com#{s3_bucket}/$(tarfile $r).gz"

        if curl -sLI $s3_url | head -n 1 | grep -q ' 200 ' ; then
          echo "----> S3 resource already exists at $s3_url, skipping..."
        else
          rm -rf $HOME/$(tarfile $r)*
          (cd /usr/local/rbenv/versions && tar cpf $HOME/$(tarfile $r) $r)
          gzip -9 $HOME/$(tarfile $r)
          s3-put -S \
            -k $(cat "$HOME/aws_access_key_id.txt") \
            -s "$HOME/aws_secret_access_key.txt" \
            -a "$HOME/aws_s3_headers.txt" \
            -T $HOME/$(tarfile $r).gz #{s3_bucket}/$(tarfile $r).gz
          echo "----> Uploaded resource to S3 at $s3_url"
        fi
      done ; unset r
    PROVISION
  end
end
