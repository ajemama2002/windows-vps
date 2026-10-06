name: Free Windows RDP
on: workflow_dispatch
jobs:
  rdp:
    runs-on: windows-latest
    timeout-minutes: 360
    steps:
      - name: Enable RDP
        run: |
          Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -value 0
          Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
          Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -value 1
          net user runneradmin Bobby123!
      - name: Install Tailscale
        run: |
          $url = "https://pkgs.tailscale.com/stable/tailscale-setup-latest.exe"
          Invoke-WebRequest -Uri $url -OutFile tailscale-setup.exe
          Start-Process -FilePath .\tailscale-setup.exe -ArgumentList "/quiet" -Wait
          Start-Sleep 10
      - name: Connect to Tailscale
        run: |
          & "C:\Program Files\Tailscale\tailscale.exe" up --authkey=${{ secrets.TAILSCALE_AUTHKEY }} --hostname=gh-win-${{ github.run_id }}
          & "C:\Program Files\Tailscale\tailscale.exe" ip -4
          & "C:\Program Files\Tailscale\tailscale.exe" status
      - name: Keep Alive
        run: |
          echo "================================"
          echo "Username: runneradmin"
          echo "Password: Bobby123!"
          echo "Check IP above - use that to connect"
          echo "================================"
          while ($true) {
            echo "[$(Get-Date)] VPS is running - Don't close this!"
            Start-Sleep 60
          }
