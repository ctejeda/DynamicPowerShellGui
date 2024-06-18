# PowerShell Data Viewer with GUI and Filtering

This PowerShell script provides a graphical user interface (GUI) to display data retrieved from various cmdlets, such as `Get-Process` or `Get-ADUser`. The GUI includes a filtering option to allow users to dynamically filter the displayed data based on their input.

## Features

- **Dynamic Data Display**: Automatically adapts to display any enumerable collection of objects.
- **Filtering Capability**: Users can filter the displayed data using a TextBox and Button.
- **Resizable GUI**: The DataGridView resizes dynamically when the form is maximized or resized.

## Prerequisites

- PowerShell 5.1 or later
- .NET Framework (Windows Forms)

## Usage

### Example with `Get-Process`

1. **Open PowerShell ISE or any PowerShell editor.**
2. **Copy and paste the script below into the editor.**
3. **Run the script.**

```powershell
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing

function Show-DataInGrid {
    param (
        [Parameter(Mandatory = $true)]
        [System.Collections.IEnumerable]$Data
    )

    # Create the form
    $form = New-Object System.Windows.Forms.Form
    $form.Text = "Data Viewer"
    $form.Size = New-Object System.Drawing.Size(800, 450)
    $form.StartPosition = "CenterScreen"

    # Create the filter TextBox
    $textBox = New-Object System.Windows.Forms.TextBox
    $textBox.Location = New-Object System.Drawing.Point(10, 10)
    $textBox.Size = New-Object System.Drawing.Size(680, 20)
    $form.Controls.Add($textBox)

    # Create the filter Button
    $button = New-Object System.Windows.Forms.Button
    $button.Location = New-Object System.Drawing.Point(700, 10)
    $button.Size = New-Object System.Drawing.Size(75, 20)
    $button.Text = "Filter"
    $form.Controls.Add($button)

    # Create the DataGridView
    $dataGridView = New-Object System.Windows.Forms.DataGridView
    $dataGridView.Location = New-Object System.Drawing.Point(10, 40)
    $dataGridView.Size = New-Object System.Drawing.Size(760, 360)
    $dataGridView.Anchor = [System.Windows.Forms.AnchorStyles]::Top -bor `
                           [System.Windows.Forms.AnchorStyles]::Bottom -bor `
                           [System.Windows.Forms.AnchorStyles]::Left -bor `
                           [System.Windows.Forms.AnchorStyles]::Right
    $dataGridView.AutoSizeColumnsMode = "Fill"
    $dataGridView.ReadOnly = $true
    $form.Controls.Add($dataGridView)

    # Create DataTable and add columns dynamically
    $dataTable = New-Object System.Data.DataTable
    $firstItem = $Data | Select-Object -First 1
    $columns = $firstItem.PSObject.Properties.Name
    foreach ($column in $columns) {
        $dataTable.Columns.Add($column) > $null
    }

    # Add rows dynamically
    function Populate-DataTable {
        $dataTable.Rows.Clear()
        foreach ($item in $Data) {
            $row = $dataTable.NewRow()
            foreach ($column in $columns) {
                $row[$column] = $item.$column
            }
            $dataTable.Rows.Add($row) > $null
        }
    }
    
    Populate-DataTable

    $dataGridView.DataSource = $dataTable

    # Handle the Resize event to adjust DataGridView size
    $form.Add_Shown({$form.Activate()})
    $form.add_Resize({
        $dataGridView.Size = New-Object System.Drawing.Size($form.ClientSize.Width - 20, $form.ClientSize.Height - 50)
    })

    # Add the filter functionality
    $button.Add_Click({
        $filter = $textBox.Text
        $filteredData = $Data | Where-Object {
            $match = $false
            foreach ($column in $columns) {
                if ($_.PSObject.Properties[$column].Value -like "*$filter*") {
                    $match = $true
                }
            }
            $match
        }
        $dataTable.Rows.Clear()
        foreach ($item in $filteredData) {
            $row = $dataTable.NewRow()
            foreach ($column in $columns) {
                $row[$column] = $item.$column
            }
            $dataTable.Rows.Add($row) > $null
        }
    })

    # Show the form
    [void] $form.ShowDialog()
}

# Example usage with Get-Process
$processes = Get-Process | Select-Object -Property Id, ProcessName, CPU, WorkingSet
Show-DataInGrid -Data $processes

# Example usage with Get-ADUser (uncomment if Active Directory module is installed)
# Import-Module ActiveDirectory
# $users = Get-ADUser -Filter * -Property DisplayName, SamAccountName, UserPrincipalName | Select-Object DisplayName, SamAccountName, UserPrincipalName
# Show-DataInGrid -Data $users
