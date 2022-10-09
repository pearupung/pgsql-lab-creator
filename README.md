# PGSQL Lab Creator

At university I often times sat in a lab behind a computer with no feeling of order - everybody paced themselves and the ones that needed help were in disarray. Managing a classroom is hard and managing a computer classroom without knowledge who needs help or who is behind is *crippling*. For me, at least. This project:

1. sets up an empty PGSQL instance that everybody can log into and complete a lab
2. sets up automatic monitoring based on some example database and gives feedback on how well the students are progressing

The live demo will be hosted as soon as I have made enough progress on it.

## Architecture

Overall, this is an architecture I have in mind:

```mermaid
graph LR;
  %% Services to set up
  ldap[LDAP server]
  pgsql[Postgres]
  exporter[SQLExporter]
  prometheus[Prometheus instance]
  grafana[Grafana]
  server[Python Flask server]
  caddy[Caddy reverse proxy]
  
  %% The docker instance that runs services
  docker[Docker instance]

  %% Participants 
  lab-master[Lab Master]
  lab-student[Lab Student]

  %% Infrastructure setup
  lab-master-- executes docker compose -->docker

  %% Infrastructure configuration
  lab-master-- enters information about the database and lab participants -->server
  server-- makes a query file and forwards it -->exporter
  server-- configures LDAP accounts -->ldap
  server-- configures panel variables and setup -->grafana
  server-- creates databases and grants access for the users -->pgsql
  lab-student-- sees initial database passwords on screen -->server

  %% Lab interaction
  lab-student-- changes LDAP password -->ldap
  lab-student-- logs in to psql instance-->pgsql
  pgsql-- authenticates against LDAP provider-->ldap
  lab-student-- starts lab-->pgsql
  lab-master-- looks at dashboard to see how people are progressing -->grafana
  lab-student-- looks at dashboard to see if anybody needs help -->grafana

  %% Infrastructure interaction
  grafana-- poll Prometheus for data to put on screen -->prometheus
  prometheus-- poll SQL Exporter -->exporter
  exporter-- poll Postgres instance -->pgsql
```

### Lab master view

The lab master is the one that sets up the infrastructure and quides people through the lab. His overall activities are infrastructure setup and configuration. His responsibilities end when the participant can access the database and then he can relax and watch the monitoring to see how everybody progresses. The procedure looks like this:

```mermaid
graph LR;
  %% Services to set up
  grafana[Grafana]
  server[Python Flask server]
  
  %% The docker instance that runs services
  docker[Docker instance]

  %% Participants 
  lab-master[Lab Master]

  %% Infrastructure setup
  lab-master-- 1. executes docker compose -->docker

  %% Infrastructure configuration
  lab-master-- 2. enters information about the database and lab participants -->server

  lab-master-- 4. looks at dashboard to see how people are progressing -->grafana
  lab-master-- 3. sees initial database passwords on screen and forwards them to students-->server
```

### Lab student view

The lab student must simply log in to the database and begin work. The student can see his progress on the monitoring screen as well as his peers' progress. Depending on the LDAP solution students can get an initial password and change it or use their university authentication source.

```mermaid
graph LR;
  %% Services to set up
  ldap[LDAP server]
  pgsql[Postgres]
  grafana[Grafana]
  server[Python Flask server]

  %% Participants 
  lab-student[Lab Student]

  %% Infrastructure setup

  %% Infrastructure configuration
  lab-student-- sees initial database passwords on screen -->server

  %% Lab interaction
  lab-student-- changes LDAP password -->ldap
  lab-student-- logs in to psql instance-->pgsql
  pgsql-- authenticates against LDAP provider-->ldap
  lab-student-- starts lab-->pgsql
  lab-student-- looks at dashboard to see if anybody needs help -->grafana
```
