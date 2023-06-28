# Api rest with Symfony 6 & php 8

## Prerequisites:

- Composer

- Symfony CLI

- MySQL

- PHP >= 8.1

## Creating Project

- Create project via composer:

```
composer create-project symfony/skeleton symfony-6-rest-api

```

- Create project via Symfony CLI

```
symfony new symfony-6-rest-api
```

## Installing Packages

The packages will ask you to confirmation, press yes.

```
composer require jms/serializer-bundle
```

```
composer require friendsofsymfony/rest-bundle
```

```
composer require symfony/maker-bundle
```

```
composer require symfony/orm-pack
```

## Configure FOSRest Bundle

Open the file config/packages/fos_rest.yaml and uncomment the following:

```ts
fos_rest:
    format_listener:
        rules:
            - { path: ^/api, prefer_extension: true, fallback_format: json, priorities: [ json, html ] }
```

## Database Configuration

Open .env file and set the database configuration. I am using MySQL in this case, so I will uncomment DATABASE_URL variable, and comment the one by default.

```c
DATABASE_URL="mysql://root:@127.0.0.1:3306/tiendapi?serverVersion=8.0.32&charset=utf8mb4"
# DATABASE_URL="postgresql://app:@127.0.0.1:5432/tiendapi?serverVersion=15&charset=utf8"
...
```

I have changed app: -> root: and removed !ChangeMe! (here you should add your db password if you have one)

And where it says app, change it for the name of the data base.

If you are using different ports, make sure you change them in this line.

## Create Entity and Migration

```
php bin/console make:entity entity-name
```

Set the field names, types and if they can be null or not.

```
php bin/console make:migration
```

This will create a migration file containing SQL. Then we will have to run it.

```
php bin/console doctrine:migrations:migrate
```

## Generate the controller

```
php bin/console make:controller entity-name
```

this will create a controller with the name of the entity, ending with Controller.

## Create API on the controller

In this example my entity is called Project

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Annotation\Route;
use Doctrine\Persistence\ManagerRegistry;
use Symfony\Component\HttpFoundation\Request;
use App\Entity\Project;

#[Route('/api', name: 'api_')]
class ProjectController extends AbstractController
{
    #[Route('/projects', name: 'project_index', methods: ['get'])]
    public function index(ManagerRegistry $doctrine): JsonResponse
    {
        $products = $doctrine
            ->getRepository(Project::class)
            ->findAll();

        $data = [];

        foreach ($products as $product) {
            $data[] = [
                'id' => $product->getId(),
                'name' => $product->getName(),
                'description' => $product->getDescription(),
            ];
        }

        return $this->json($data);
    }


    #[Route('/projects', name: 'project_create', methods: ['post'])]
    public function create(ManagerRegistry $doctrine, Request $request): JsonResponse
    {
        $entityManager = $doctrine->getManager();

        $databody = $request->toArray();
        $project = new Project();
        $project->setName($databody["name"]);
        $project->setDescription($databody["description"]);

        $entityManager->persist($project);
        $entityManager->flush();

        $data = [
            'id' => $project->getId(),
            'name' => $project->getName(),
            'description' => $project->getDescription(),
        ];

        return $this->json($data);
    }


    #[Route('/projects/{id}', name: 'project_show', methods: ['get'])]
    public function show(ManagerRegistry $doctrine, int $id): JsonResponse
    {
        $project = $doctrine->getRepository(Project::class)->find($id);

        if (!$project) {

            return $this->json('No project found for id ' . $id, 404);
        }

        $data = [
            'id' => $project->getId(),
            'name' => $project->getName(),
            'description' => $project->getDescription(),
        ];

        return $this->json($data);
    }

    #[Route('/projects/{id}', name: 'project_update', methods: ['put', 'patch'])]
    public function update(ManagerRegistry $doctrine, Request $request, int $id): JsonResponse
    {
        $entityManager = $doctrine->getManager();
        $project = $entityManager->getRepository(Project::class)->find($id);

        if (!$project) {
            return $this->json('No project found for id' . $id, 404);
        }
        $databody = $request->toArray();

        $project->setName($databody["name"]);
        $project->setDescription($databody["description"]);
        $entityManager->flush();

        $data = [
            'id' => $project->getId(),
            'name' => $project->getName(),
            'description' => $project->getDescription(),
        ];

        return $this->json($data);
    }

    #[Route('/projects/{id}', name: 'project_delete', methods: ['delete'])]
    public function delete(ManagerRegistry $doctrine, int $id): JsonResponse
    {
        $entityManager = $doctrine->getManager();
        $project = $entityManager->getRepository(Project::class)->find($id);

        if (!$project) {
            return $this->json('No project found for id' . $id, 404);
        }

        $entityManager->remove($project);
        $entityManager->flush();

        return $this->json('Deleted a project successfully with id ' . $id);
    }
}
```

## Run the API

```
symfony server:start
```

You can try it with Postman
