# ansible-productivity-apps

Ansible playbooks that make productivity apps match a declared configuration. Each playbook loads one private
declaration document at run time and passes its sections to the roles of a public collection. The first app is
JetBrains YouTrack, configured through the roles of the `jpe_ansible.youtrack` collection.
