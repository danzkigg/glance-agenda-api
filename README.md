<div align="center">

# Glance Agenda API

![README](https://img.shields.io/badge/Actively%20Maintained-Green)
![README](https://img.shields.io/github/v/release/danzkigg/glance-agenda-api)
![README](https://img.shields.io/docker/pulls/danzkigg/glance-agenda-api)

A simple Docker service that fetches calendar data from Sonarr and Radarr, merges and sorts it, and provides a single, unified API endpoint for the Glance widget.
This allows you to see all your upcoming TV shows and movies in one chronological list.

___

<img width="537" height="416" alt="Screenshot 2025-08-20 at 23 41 31" src="https://github.com/user-attachments/assets/479c2bd4-8e14-4cc6-9e27-9945eabed699" />

</div>

## Prerequisites

* **Docker:** [Install Docker](https://docs.docker.com/engine/install/)
* **Docker Compose:** [Install Docker Compose](https://docs.docker.com/compose/install/)

---

## Quick Start Guide

You can get this service running in 3 simple steps.

### Step 1: Create a Folder

Create a new folder on your server to hold the configuration files.

```bash
mkdir glance-agenda
cd glance-agenda
```

### Step 2: Create Configuration Files

Inside the `glance-agenda` folder you just created, create the following file.

**`docker-compose.yml`** 

```yaml
services:
  agenda-api:
    image: danzkigg/glance-agenda-api:latest
    container_name: glance-agenda-api
    restart: unless-stopped
    ports:
      # Exposes the API on port 5000.
      # You can change the first number if that port is already in use.
      # Example: "4000:5000"
      - "5000:5000"
    environment:
      SONARR_URL: http://<SONARR_HOST>:8989
      SONARR_API_KEY: your_sonarr_api_key
      SONARR_DAYS_AHEAD: 90
      RADARR_URL: http://<RADARR_HOST>:7878
      RADARR_API_KEY: your_radarr_api_key
      RADARR_DAYS_AHEAD: 365
```

### Step 3: Run It!

Open your terminal, make sure you are in the `glance-agenda` folder, and run the service:

```bash
docker compose up -d
```

The API will now be running and accessible at `http://<your-server-ip>:5000/api/agenda`.

---

## Glance Widget Configuration

Use the following configuration for your `custom-api` widget in Glance.

**Important:** Change the `url` to point to the IP address of the machine where you are running the Docker container.

```yaml
    - type: custom-api
      title: Agenda
      url: http://<IP-DOCKER>:5000/api/agenda
      cache: 1h
      options:
        collapse_after: 12 # Control the "Show More" button here. Use -1 to disable.
      template: |
        {{ $collapseAfter := .Options.IntOr "collapse_after" -1 }}
        {{ $today := (offsetNow "0h" | formatTime "2006-01-02") }}
  
        {{ if gt $collapseAfter 0 }}
        <div class="list collapsible-container" data-collapse-after="{{ $collapseAfter }}" style="--list-gap: 8px;">
        {{ else }}
        <div class="list" style="--list-gap: 8px;">
        {{ end }}
  
        {{/* Loop over the top-level array directly */}}
        {{ range .JSON.Array "" }}
          {{ $date := .String "date" }}
  
          {{/* Date Header */}}
          <div class="size-sm color-paragraph" style="padding-top: 15px; font-weight: bold;">
            {{ if eq $date $today }}
              Today
            {{ else }}
              {{ $date | parseTime "2006-01-02" | formatTime "Jan 2" }}
            {{ end }}
          </div>
                  
          {{/* Loop over the "items" array for each date group */}}
          {{ range .Array "items" }}
            <div class="flex items-center" style="gap: 15px;">
              <div style="width: 50px; text-align: right; flex-shrink: 0;">
                {{ if .Bool "has_file" }}
                  <span class="color-positive" style="display: inline-flex; align-items: center;">
                    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" style="width: 1.4em; height: 1.4em;"><path fill-rule="evenodd" d="M2.25 12c0-5.385 4.365-9.75 9.75-9.75s9.75 4.365 9.75 9.75-4.365 9.75-9.75 9.75S2.25 17.385 2.25 12zm13.36-1.814a.75.75 0 10-1.22-.872l-3.236 4.53L9.53 12.22a.75.75 0 00-1.06 1.06l2.25 2.25a.75.75 0 001.14-.094l3.75-5.25z" clip-rule="evenodd" /></svg>
                  </span>
                {{ else if eq $date $today }}
                  <span class="color-highlight">
                    {{ .String "release_datetime" | parseTime "rfc3339" | formatTime "15:04" }}
                  </span>
                {{ else }}
                  <span class="color-paragraph">•</span>
                {{ end }}
              </div>
              <div class="flex items-center" style="gap: 8px;">
                <span class="color-theme-contrast-30" style="flex-shrink: 0;">
                <div>
                  <span class="color-paragraph">{{ .String "title" }}</span>
                  <span class="size-sm color-theme-contrast-50">{{ .String "details" }}</span>
                </div>
              </div>
            </div>
          {{ end }}
        {{ end }}
        </div>
```

---

## Updating the Service

To update to the latest version of the API image, run these commands from your `glance-agenda` folder:

```bash
# Pull the latest image from Docker Hub
docker compose pull

# Restart the container with the new image
docker compose up -d
```

## Contributors
<a href="https://github.com/danzkigg/glance-agenda-api/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=danzkigg/glance-agenda-api" />
</a>
