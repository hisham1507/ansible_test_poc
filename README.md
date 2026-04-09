# Mainframe Ansible Introduction Playbook

## Overview

This is a template Ansible playbook designed to introduce Mainframe Subject Matter Experts (SMEs) to Ansible automation. The playbook demonstrates basic Ansible concepts and patterns using simple z/OS tasks that are familiar to mainframe professionals.

## Purpose

This playbook serves as a hands-on learning tool to help Mainframe SMEs:
- Understand Ansible's declarative automation approach
- Learn how to structure playbooks, roles, and inventory
- Execute basic z/OS operations using IBM's z/OS Ansible collections
- Bridge the gap between traditional mainframe operations and modern automation

## Prerequisites

Before running this playbook, ensure you have:

1. **Ansible Automation Platform (AAP) Environment**
   - Access to Ensono's Ansible Automation Platform
   - `ansible-navigator` installed on your control node
   - Access to the Ensono execution environment registry
   
2. **z/OS Target System**
   - SSH access configured
   - IBM Z Open Automation Utilities (ZOAU) installed
   - Python for z/OS installed
   - Appropriate user credentials

3. **Execution Environment**
   - The Ensono AAP execution environment (`aap-25-ensono-ee`) includes:
     - Ansible Core 2.15+
     - IBM z/OS Core Collection (`ibm.ibm_zos_core`)
     - All required dependencies and Python libraries
   - Available at: `hub.ansible.envision.ensono.com/aap-25-ensono-ee`

## Project Structure

```
.
├── inventory.yml              # Defines target mainframe systems
├── site.yml                   # Main playbook entry point
├── roles/
│   └── hello_world/          # Example role
│       └── tasks/
│           └── main.yml      # Tasks for gathering host info
└── README.md                 # This file
```

## Configuration

### Inventory File

The [inventory.yml](inventory.yml) file defines your z/OS target system. Update the following values:

```yaml
all:
  hosts:
    LAB1:
      ansible_host: your.mainframe.hostname  # Update with your z/OS LPAR hostname/IP
      ansible_port: 922                       # SSH port (typically 22 or 922)
```

Key z/OS-specific variables:
- `ZOAU`: Path to Z Open Automation Utilities
- `PYZ`: Path to Python installation on z/OS
- `environment_vars`: Environment settings required for z/OS Ansible execution

### Credentials

This playbook uses **Envision Device Console** for credential management at runtime. Credentials are automatically retrieved based on:

- **`CMDB_Client`**: The client identifier (e.g., "Ensono Shared")
- **Inventory hostname**: Used to lookup the appropriate credentials

```yaml
LAB1:
  CMDB_Client: "Ensono Shared"  # Identifies which credential set to use
```

**How it works:**
1. Ansible Automation Platform reads the `CMDB_Client` variable from your inventory
2. Combines it with the inventory hostname (e.g., "LAB1")
3. Retrieves credentials from Envision Device Console automatically
4. No need to provide username/password manually

**For more details**, see the [Envision Credentials documentation](https://techdocs.ensono.com/applications/ansible/Host%20Connectivity/0-Envision-Credentials/index.html).

**Note:** This credential lookup is handled by the Ensono AAP environment and execution environment automatically.

## Running the Playbook

### Using Ansible Navigator (Recommended)

This playbook is designed to run with **ansible-navigator** using Ensono's AAP execution environment:

```bash
ansible-navigator run site.yml -i inventory.yml -m stdout --eei hub.ansible.envision.ensono.com/aap-25-ensono-ee
```

Flags explained:
- `run site.yml`: Executes the playbook
- `-i inventory.yml`: Specifies the inventory file
- `-m stdout`: Display mode (stdout shows output directly in terminal)
- `--eei`: Execution Environment Image from Ensono's registry

### With Verbose Output

For learning purposes, you may want to see detailed output:

```bash
ansible-navigator run site.yml -i inventory.yml -m stdout --eei hub.ansible.envision.ensono.com/aap-25-ensono-ee -v
```

Additional verbosity levels: `-v`, `-vv`, `-vvv`, `-vvvv`

### Alternative: Using ansible-playbook (Local Development)

If you're testing locally without execution environments:

```bash
ansible-playbook -i inventory.yml site.yml -u YOUR_USERNAME -k
```

**Note:** Ensure you have the `ibm.ibm_zos_core` collection installed locally.

## What This Playbook Does

### 1. Hello World Role
- Gathers facts about the z/OS system
- Displays connection details including:
  - Hostname
  - Operating system distribution
  - Connected user ID
  - Connection method

### 2. z/OS Ping
- Executes the `zos_ping` module
- Verifies connectivity and proper configuration
- Confirms IBM z/OS collection is working

## Expected Output

When successful, you should see output similar to:

```
PLAY [Ensono Sample Playbook] *********************************************

TASK [Run Hello World to the LPAR] ****************************************

TASK [hello_world : Gathering Host Facts] *********************************
ok: [LAB1]

TASK [hello_world : Host Connection Details] ******************************
ok: [LAB1] => {
    "msg": [
        "Connected host is YOUR_HOSTNAME.",
        "OS family is ZVSE.",
        "Connected using YOUR_USER.",
        "Connection Method - Direct"
    ]
}

TASK [Run z/OS ping] ******************************************************
ok: [LAB1]

PLAY RECAP ****************************************************************
LAB1                       : ok=3    changed=0    unreachable=0    failed=0
```

## Learning Concepts

This simple playbook introduces several key Ansible concepts:

### 1. **Execution Environments**
Containerized runtimes that include Ansible, collections, and dependencies. Ensures consistency across different systems.

### 2. **Inventory**
Defines "what" you're automating - your mainframe systems and their connection details.

### 3. **Playbook** 
Defines "what should happen" - the sequence of tasks to execute ([site.yml](site.yml)).

### 4. **Roles**
Reusable, modular collections of tasks ([hello_world](roles/hello_world)).

### 5. **Tasks**
Individual automation actions executed on target systems.

### 6. **Modules**
Pre-built automation units (like `zos_ping`, `setup`).

### 7. **Variables**
Configuration values that can be reused across playbooks.

### 8. **Facts**
System information automatically gathered by Ansible.

## Next Steps

After successfully running this playbook, you can:

1. **Modify the hello_world role** to add your own tasks
2. **Explore IBM z/OS Core Collection modules** for common mainframe operations:
   - `zos_job_submit`: Submit JCL jobs
   - `zos_data_set`: Manage datasets
   - `zos_copy`: Copy files to/from z/OS
   - `zos_operator`: Issue operator commands
   
3. **Create new roles** for your specific automation needs
4. **Add more hosts** to your inventory
5. **Implement error handling** with blocks and rescue
6. **Use templates** for generating JCL or configuration files

## Troubleshooting

### Connection Issues
- Verify SSH connectivity: `ssh -p 922 username@mainframe.host`
- Check firewall rules and network access
- Confirm SSH daemon is running on z/OS

### ZOAU/Python Path Issues
- Verify paths in inventory match your z/OS installation
- Check with your z/OS administrator for correct paths
- Ensure Python and ZOAU are properly installed

### Module Not Found
- Ensure you're using the correct execution environment: `--eei hub.ansible.envision.ensono.com/aap-25-ensono-ee`
- The execution environment includes all required collections
- For local development, install IBM z/OS collection: `ansible-galaxy collection install ibm.ibm_zos_core`

### Permission Errors
- Ensure your user has necessary RACF/ACF2/Top Secret permissions
- Check file and dataset access permissions

## Ensono Resources

### Internal Documentation
- **Ansible Community of Practice**: [Writing Playbook](https://techdocs.ensono.com/applications/ansible/Community%20of%20Practice/Writing-a-Playbook/index.html)
  - Comprehensive guides for Ansible creators at Ensono
  - Best practices and standards for playbook development
  - Templates and examples

### Collaboration
- **Ensono Ansible Teams Channel**: [Join the conversation](https://teams.microsoft.com/l/team/19%3AbYYSGSUmQKgIKD7_wf83vMcqBfPewF33gd9mtxZLDgU1%40thread.tacv2/conversations?groupId=72c801d0-974f-4d0f-9f2f-cda91641c586&tenantId=d483ed84-f7bf-4078-a58f-fb250feccf8f)
  - Ask questions and get support from the Ansible team
  - Share experiences and learn from colleagues
  - Stay updated on Ansible-related announcements

## Additional Resources

- **Ansible Documentation**: https://docs.ansible.com/
- **Ansible Navigator Guide**: https://ansible.readthedocs.io/projects/navigator/
- **IBM z/OS Core Collection**: https://galaxy.ansible.com/ibm/ibm_zos_core
- **IBM z/OS Ansible Docs**: https://ibm.github.io/z_ansible_collections_doc/
- **Red Hat Ansible for IBM Z**: https://www.redhat.com/en/technologies/management/ansible/ibm-z

## Support

For questions or issues with this template:
1. Review the playbook execution output with verbose mode (`-vvv`)
2. Check the [Ensono Ansible Community of Practice](https://techdocs.ensono.com/applications/ansible/Community%20of%20Practice/Writing-a-Playbook/index.html) documentation
3. Ask questions in the [Ensono Ansible Teams Channel](https://teams.microsoft.com/l/team/19%3AbYYSGSUmQKgIKD7_wf83vMcqBfPewF33gd9mtxZLDgU1%40thread.tacv2/conversations?groupId=72c801d0-974f-4d0f-9f2f-cda91641c586&tenantId=d483ed84-f7bf-4078-a58f-fb250feccf8f)
4. Consult with your Ansible or mainframe automation team
5. Review Ansible and IBM z/OS collection documentation

## License

This is a template playbook for educational purposes.

---

**Welcome to Mainframe Automation with Ansible!** 🚀
