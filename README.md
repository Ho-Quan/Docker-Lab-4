# Docker Lab 4

## Publishing Ports 
### To expose ports and run a container
```
docker run -d -p 8080:80 docker/welcome-to-docker
```
### Verify the port
```
http://localhost:8080/
```
## Overriding Container Defaults
### Running Multiple Instances of Postgres
- First Postgres Container
```
docker run -d -e POSTGRES_PASSWORD=secret -p 5432:5432 postgres
```
- Second Postgres Container on a different port
```
docker run -d -e POSTGRES_PASSWORD=secret -p 5433:5432 postgres
```
### Running Postgres in a Controlled Network
- Create a custom network
```
docker network create mynetwork
```
- Run Postgres on a custom network:
```
docker run -d -e POSTGRES_PASSWORD=secret -p 5434:5432 --network mynetwork postgres
```
- Verify the network
```
docker network ls
```
### Managing Resource Allocation (Memory and CPU)
- Command to limit memory and CPU
```
docker run -d -e POSTGRES_PASSWORD=secret --memory="512m" --cpus=".5" postgres
```
- To monitor read time usage
```
 docker network ls
 ```
### Override CMD and ENTRYPOINT in Docker Compose
- Make sure to update the docker-compose.yml file to define the Postgres service and override the entry point
- Start the container with custom entry point
```
docker compose up -d
```
- Go into the Postgres database
```
psql -U postgres
```
### Override CMD and ENTRYPOINT Directly with ```docker run```
- Command to override entrypoint
```
docker run -e POSTGRES_PASSWORD=secret postgres docker-entrypoint.sh -h localhost -p 5432
```
## Persisting Container Data
- Create a volume
```
docker volume create postgres_data
```
- Attach the volume to a container
```
docker run --name=db -e POSTGRES_PASSWORD=secret -d -v postgres_data:/var/lib/postgresql/data postgres
```
- Access the running container
```
docker exec -ti db psql -U postgres
```
- Create a table and insert data
```
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    description VARCHAR(100)
);
INSERT INTO tasks (description) VALUES ('Finish work'), ('Have fun');
```
- Stop and remove the container
```
docker stop db
docker rm db
```
- Start a new container with the same volume
```
docker run --name=new-db -d -v postgres_data:/var/lib/postgresql/data postgres
```
- Remove the volume
```
docker volume rm postgres_data
```
- To remove all unused volumes
```
docker volume prune
```
## Sharing Local Files with Containers
- Run an HTTPD container
```
docker run -d -p 8080:80 --name my_site httpd:2.4
```
- Verify by accessing
```
http://localhost:8080/
```
- Delete the container
```
docker rm -f my_site
```
- Create a directory and create an index.html within that folder
```
mkdir public_html
```
- Start the container with a bind mount
```
docker run -d --name my_site -p 8080:80 -v $(pwd)/public_html:/usr/local/apache2/htdocs/ httpd:2.4
```
- Remove the file and container
```
rm public_html/index.html
docker rm -f my_site
```
## Multi-Container Applications
- Build images for the containers
```
docker build -t nginx .
docker build -t web .
```
- Create a network
```
docker network create sample-app
```
- Run Redis container
```
docker run -d --name redis --network sample-app --network-alias redis redis
```
- Run the first web container
```
docker run -d --name web1 -h web1 --network sample-app --network-alias web1 web
```
- Run the second web container
```
docker run -d --name web2 -h web2 --network sample-app --network-alias web2 web
```
- Run the Nginx container
```
docker run -d --name nginx --network sample-app -p 80:80 nginx
```
- Verify all containers are running
```
docker ps
```
- Use Docker Compose to bring up the application
```
docker compose up -d --build
```
- Delete the containers
```
docker compose down
```
