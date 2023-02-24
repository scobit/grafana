#### search for dashboards with aninymous access
curl http://localhost:3000/api/search

#### search for dashboards with basic authentication
curl http://admin:admin@localhost:3000/api/search

#### delete dashboard by UID using basic authentication
curl -X DELETE "http://admin:admin@localhost:3000/api/dashboards/uid/W5KDrdKnz"
