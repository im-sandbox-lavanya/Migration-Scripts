
## Migration from GHEC to GHEC/GHEC-EMU

This document provides guidelines and steps for migrating from GitHub Enterprise Cloud (GHEC) to GHEC-EMU. 
Follow the instructions carefully to ensure a smooth migration process.


### Prerequisites

Before starting the migration process, ensure that you have set the following environment variables:

- `GH_SOURCE_PAT`: Personal Access Token for the source organization.
- `GH_PAT`: Personal Access Token for the target organization.

You can set these environment variables in your terminal session as follows:

```bash
bash
export GH_SOURCE_PAT=your_source_pat
export GH_PAT=your_target_pat
```
```powershell
powershell
$env:GH_PAT="TOKEN"
$env:GH_SOURCE_PAT="TOKEN"
```

Make sure these tokens have the necessary permissions to access the repositories and perform the migration tasks.

### Install the GEI Extension

Before generating the migration script, you need to install the GitHub Enterprise Importer (GEI) extension. This extension provides the necessary tools for automating the migration process.

To install the GEI extension, run the following command:

```powershell
gh extension install github/gh-gei
```

Ensure that the installation completes successfully. You can verify the installation by running:

```powershell
gh gei --help
```

This command should display the help information for the GEI extension, confirming that it is installed and ready to use.


### Step 1: Execute the `im-checkpat.py` Script

Run the `im-checkpat.py` script for both the source Personal Access Token (PAT) and the target PAT. 
This script will check the access and available organizations for these PATs.

```powershell
python im-checkpat.py --token TOKEN
```

Ensure that the script completes successfully and verifies the access permissions and organizations for both PATs.


### Step 2: Run the `im-inventory.py` Script on Source Org

Execute the `im-inventory.py` script to generate the inventory file for both the source organization before migration and name it as a pre-migration inventory file. 

```powershell
python im-inventory.py --source_org <org_name> --output_file <filename> --workers 8
```

This script will create an inventory of repositories, in both organizations, which is essential for planning the migration.

### Step 3: Run the `im-inventory.py` Script on Target Org

Execute the `im-inventory.py` script to generate the inventory file for both the source organization before migration and name it as a pre-migration inventory file. 

```powershell
python im-inventory.py --source_org <target_org> --output_file <filename> --workers 8
```

This script will create an inventory of repositories, in both organizations, which is essential for planning the migration.


### Step 4: Run the `im-inventory-filter-repos.py` Script for Selective Repositories

If you need to generate an inventory for selective repositories, use the `im-inventory-filter-repos.py` script. This script allows you to specify a list of repositories to include in the inventory.

```powershell
python im-inventory-filter-repos.py --source_org <org_name> --output_file <filename> --workers 8 --filter_file <repo_list.csv>
```

Replace `<repo_list>` with a comma-separated list of repository names you want to include in the inventory. This step is useful for focusing on specific repositories during the migration process.

### Step 5: Review the Inventory File

After running the `im-inventory.py` script, review the generated inventory file for the source organization. This file contains a list of repositories and their details, which will help in planning the migration process.

Ensure that all repositories are listed and that the details are accurate. This step is crucial for identifying any potential issues before proceeding with the migration.

### Step 6: Generate the Migration Script Using GEI

Use the GEI (GitHub Enterprise Importer) to generate the migration script. This tool helps automate the migration process by creating scripts based on the inventory file.

```powershell
gh gei generate-script --github-source-org <source_org> --github-target-org <target_org> -output <migration_script.ps>
```

Replace `<source_org>`, `<target_org>`, `<filename>`, and `<migration_script.sh>` with the appropriate values. The generated script will contain the necessary commands to migrate repositories from the source organization to the target organization.

Ensure that the script is generated successfully and review it to confirm that it includes all the required steps for the migration.


### Step 7: Execute the Migration Script

After generating the migration script in the previous step, execute the script to start the migration process. Ensure that you have reviewed the script and made any necessary adjustments before running it.

To execute the migration script, run the following command:

```powershell
bash migration_script.ps
```

Replace `migration_script.ps` with the name of the script generated in the previous step. This script will perform the migration of repositories from the source organization to the target organization.

Monitor the output of the script to ensure that the migration process completes successfully. Address any errors or issues that arise during the execution to ensure a smooth migration.

Once the script has finished running, verify that the repositories have been successfully migrated to the target organization. This may involve checking the repositories in the target organization to confirm that all data has been transferred correctly.


### Step 8: Run Inventories Post-Migration

After the migration process is complete, run the `im-inventory.py` script again on both the source and target organizations to generate post-migration inventory files. This will help you verify that all repositories have been successfully migrated.

```powershell
python im-inventory.py --source_org <source_org> --output_file <post_migration_source_inventory> --workers 8
python im-inventory.py --source_org <target_org> --output_file <post_migration_target_inventory> --workers 8
```

Replace `<source_org>`, `<target_org>`, `<post_migration_source_inventory>`, and `<post_migration_target_inventory>` with the appropriate values. 

Compare the pre-migration and post-migration inventory files to ensure that all repositories and their data have been accurately transferred. This step is crucial for confirming the success of the migration and identifying any discrepancies that need to be addressed.


### Step 9: Archive Repositories in the Source Organization Post-Migration (If Required)

After verifying the successful migration of repositories, you may choose to archive the repositories in the source organization. Archiving repositories ensures that they are preserved in a read-only state and prevents any further changes.

To archive repositories, follow these steps:

1. Identify the repositories in the source organization that need to be archived.
2. Use the `archive-repos.py` script to archive each repository. Run the following command:

    ```powershell
    python archive-repos.py
    ```
    Ensure that you replace placeholder values such as `Github_token` and `repolist.xls` with actual values. This step is crucial for the scripts to function correctly.

3. Verify that the repositories have been successfully archived by checking their status in the GitHub UI or using the GitHub API.

Archiving repositories in the source organization helps maintain a clean and organized environment while preserving the history and data of the migrated repositories.

### Reclaiming Mannequins

After the migration, you may need to reclaim mannequins (placeholder accounts for users who have left the organization) to ensure that contributions are correctly attributed to the appropriate users.

To reclaim mannequins, follow these steps:

1. Identify the mannequins in the target organization. You can use the GitHub API or the GitHub UI to find these accounts.
2. Use the `gh` CLI to reclaim mannequins. Run the following command:

    ```powershell
    gh gei reclaim-mannequin --github-target-org <target_org> --csv <filename.csv> --skip-invitation
    ```

    Replace `<target_org>` with the name of your target organization and `<filename.csv>` with the mapping file for usernames and mannequins.

3. Verify that the mannequins have been successfully reclaimed and that contributions are correctly attributed.

Reclaiming mannequins ensures that the history and contributions of former members are preserved and accurately reflected in the target organization.

