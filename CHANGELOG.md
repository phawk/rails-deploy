# 9th Feb 2015

- Add configurable cron jobs to the server
- Add libffi-dev for installing ruby 2.2.0
- Allow multiple SSH keys to be added to the server rather than just one
- nginx tweaks for large cookies
- Install imagemagick by default
- Some passenger configuration for `passenger_min_instances`, `passenger_pre_start` and `passenger_max_pool_size`
- Ensure `HTTP_X_FORWARDED_FOR` and `HTTP_X_REAL_IP` headers are set

# 30th Jan 2015

- Tuning PostgreSQL to optimise for a 1GB vps server
- Add swapfile to server
- Setup basic UFW rules
- Setup fail2ban

# 2nd Jan 2015

Initial launch

- Deploys rails servers with rbenv, node and postgres
