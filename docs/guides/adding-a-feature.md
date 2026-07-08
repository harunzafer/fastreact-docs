---
description: "FastReact tutorials - Step-by-step guides for adding new entities, integrating APIs, implementing features, and extending the FastAPI backend and React Router v7 frontend."
keywords: "fastreact tutorials, fastapi tutorial, react router v7 tutorial, add new entity, database migration, api integration, fullstack development"
---

# Tutorials

This section provides step-by-step guides for extending [FastReact](https://fastreact.dev) with new functionality.

## Adding a New Entity (End-to-End)

This tutorial walks through adding a complete "Projects" feature to demonstrate how to extend every layer of the FastReact stack. You'll learn to add database tables, backend APIs, and frontend interfaces.

### Overview

We'll build a Projects feature that allows users to:

- Create, read, update, and delete projects
- Associate projects with the current organization
- Display projects in a list and detail view

### Step 1: Database Schema

First, create a new migration for the projects table.

```bash
cd backend/db
./sqitch.sh add projects -n "Add projects table"
```

Edit the generated migration file `backend/db/deploy/projects.sql`:

```sql
-- Deploy {schema_name}:projects to pg
-- Replace {schema_name} with your actual schema name (set during init.py)

BEGIN;

CREATE TABLE IF NOT EXISTS {schema_name}.project (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    organization_id INTEGER NOT NULL REFERENCES {schema_name}.organization(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES {schema_name}."user"(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_project_organization ON {schema_name}.project(organization_id);
CREATE INDEX IF NOT EXISTS idx_project_user ON {schema_name}.project(user_id);

COMMIT;
```

Create the revert script `backend/db/revert/projects.sql`:

```sql
-- Revert {schema_name}:projects from pg

BEGIN;

DROP INDEX IF EXISTS {schema_name}.idx_project_user;
DROP INDEX IF EXISTS {schema_name}.idx_project_organization;
DROP TABLE IF EXISTS {schema_name}.project;

COMMIT;
```

Apply the migration:

```bash
./sqitch.sh dev deploy
```

### Step 2: Backend Models

Create the Pydantic models in `backend/app/model/project.py`:

```python
from datetime import datetime
from pydantic import BaseModel


class ProjectEntity(BaseModel):
    id: int
    name: str
    description: str | None = None
    organization_id: int
    user_id: int
    created_at: datetime
    updated_at: datetime


class CreateProjectRequest(BaseModel):
    name: str
    description: str | None = None


class UpdateProjectRequest(BaseModel):
    name: str | None = None
    description: str | None = None


class ProjectResponse(BaseModel):
    id: int
    name: str
    description: str | None = None
    created_at: datetime
    updated_at: datetime
```

### Step 3: Repository Layer

Create `backend/app/data/repo/project_repo.py`:

```python
from app.data.repo.base_repo import BaseRepo
from app.model.project import ProjectEntity


class ProjectRepo(BaseRepo):
    async def create_project(
        self,
        name: str,
        description: str | None,
        user_id: int,
        organization_id: int
    ) -> ProjectEntity:
        query = f"""
            INSERT INTO {self.schema}.project (name, description, user_id, organization_id)
            VALUES ($1, $2, $3, $4)
            RETURNING id, name, description, organization_id, user_id, created_at, updated_at
        """
        row = await self.fetch_one(query, name, description, user_id, organization_id)
        return ProjectEntity(**row)

    async def get_project_by_id(self, project_id: int, organization_id: int) -> ProjectEntity | None:
        query = f"""
            SELECT id, name, description, organization_id, user_id, created_at, updated_at
            FROM {self.schema}.project
            WHERE id = $1 AND organization_id = $2
        """
        row = await self.fetch_one(query, project_id, organization_id)
        return ProjectEntity(**row) if row else None

    async def get_projects_by_organization(self, organization_id: int) -> list[ProjectEntity]:
        query = f"""
            SELECT id, name, description, organization_id, user_id, created_at, updated_at
            FROM {self.schema}.project
            WHERE organization_id = $1
            ORDER BY created_at DESC
        """
        rows = await self.fetch_all(query, organization_id)
        return [ProjectEntity(**row) for row in rows]

    async def update_project(
        self,
        project_id: int,
        name: str | None,
        description: str | None,
        organization_id: int
    ) -> ProjectEntity | None:
        query = f"""
            UPDATE {self.schema}.project
            SET name = COALESCE($3, name),
                description = COALESCE($4, description),
                updated_at = now()
            WHERE id = $1 AND organization_id = $2
            RETURNING id, name, description, organization_id, user_id, created_at, updated_at
        """
        row = await self.fetch_one(query, project_id, organization_id, name, description)
        return ProjectEntity(**row) if row else None

    async def delete_project(self, project_id: int, organization_id: int) -> None:
        query = f"DELETE FROM {self.schema}.project WHERE id = $1 AND organization_id = $2"
        await self.execute(query, project_id, organization_id)
```

### Step 4: Service Layer

Create `backend/app/service/project_service.py`:

```python
from app.data.repo.project_repo import ProjectRepo
from app.model.project import CreateProjectRequest, ProjectEntity, UpdateProjectRequest


class ProjectService:
    def __init__(self, project_repo: ProjectRepo):
        self.project_repo = project_repo

    async def create_project(
        self,
        organization_id: int,
        user_id: int,
        data: CreateProjectRequest
    ) -> ProjectEntity:
        return await self.project_repo.create_project(
            data.name, data.description, user_id, organization_id
        )

    async def list_projects(self, organization_id: int) -> list[ProjectEntity]:
        return await self.project_repo.get_projects_by_organization(organization_id)

    async def get_project(
        self,
        project_id: int,
        organization_id: int
    ) -> ProjectEntity | None:
        return await self.project_repo.get_project_by_id(project_id, organization_id)

    async def update_project(
        self,
        project_id: int,
        organization_id: int,
        data: UpdateProjectRequest
    ) -> ProjectEntity | None:
        return await self.project_repo.update_project(
            project_id, data.name, data.description, organization_id
        )

    async def delete_project(self, project_id: int, organization_id: int) -> None:
        await self.project_repo.delete_project(project_id, organization_id)
```

### Step 5: API Routes

Create `backend/app/api/route/project_route.py`:

```python
from app.api.middleware.auth_handler import min_role_required
from app.config.container import Container
from app.exception.common_exception import ResourceNotFound
from app.model.project import CreateProjectRequest, ProjectResponse, UpdateProjectRequest
from app.model.role_model import Role
from app.model.user_model import CurrentUser
from app.service.project_service import ProjectService
from dependency_injector.wiring import Provide, inject
from fastapi import APIRouter, Depends

router = APIRouter()


@router.post("", response_model=ProjectResponse, operation_id="createProject")
@inject
async def create_project(
    data: CreateProjectRequest,
    user: CurrentUser = Depends(min_role_required(Role.MEMBER)),
    project_service: ProjectService = Depends(Provide[Container.project_service]),
):
    project = await project_service.create_project(
        user.organization_id, user.id, data
    )
    return ProjectResponse.model_validate(project.model_dump())


@router.get("", response_model=list[ProjectResponse], operation_id="listProjects")
@inject
async def list_projects(
    user: CurrentUser = Depends(min_role_required(Role.MEMBER)),
    project_service: ProjectService = Depends(Provide[Container.project_service]),
):
    projects = await project_service.list_projects(user.organization_id)
    return [ProjectResponse.model_validate(p.model_dump()) for p in projects]


@router.get("/{project_id}", response_model=ProjectResponse, operation_id="getProject")
@inject
async def get_project(
    project_id: int,
    user: CurrentUser = Depends(min_role_required(Role.MEMBER)),
    project_service: ProjectService = Depends(Provide[Container.project_service]),
):
    project = await project_service.get_project(project_id, user.organization_id)
    if not project:
        raise ResourceNotFound("project", project_id)
    return ProjectResponse.model_validate(project.model_dump())


@router.put("/{project_id}", response_model=ProjectResponse, operation_id="updateProject")
@inject
async def update_project(
    project_id: int,
    data: UpdateProjectRequest,
    user: CurrentUser = Depends(min_role_required(Role.MEMBER)),
    project_service: ProjectService = Depends(Provide[Container.project_service]),
):
    project = await project_service.update_project(
        project_id, user.organization_id, data
    )
    if not project:
        raise ResourceNotFound("project", project_id)
    return ProjectResponse.model_validate(project.model_dump())


@router.delete("/{project_id}", status_code=204, operation_id="deleteProject")
@inject
async def delete_project(
    project_id: int,
    user: CurrentUser = Depends(min_role_required(Role.MEMBER)),
    project_service: ProjectService = Depends(Provide[Container.project_service]),
):
    project = await project_service.get_project(project_id, user.organization_id)
    if not project:
        raise ResourceNotFound("project", project_id)
    await project_service.delete_project(project_id, user.organization_id)
```

### Step 6: Dependency Injection Setup

Update `backend/app/config/container.py` to include the new components:

```python
# 1. Add to imports at the top of the file
from app.data.repo.project_repo import ProjectRepo
from app.service.project_service import ProjectService

# 2. Add to the Container class in the repositories section
project_repo = providers.Singleton(ProjectRepo, db_config=db_config)

# 3. Add to the Container class in the services section
project_service = providers.Factory(
    ProjectService,
    project_repo=project_repo,
)
```

### Step 7: Register Routes

Update `backend/app/api/router.py` to include the new routes:

```python
# 1. Add to imports at the top
from app.api.route.project_route import router as project_router

# 2. Add to include_all_routers() function
app.include_router(project_router, prefix="/projects", tags=["Projects"])
```

### Step 8: Test the Backend API

Start the backend and verify the endpoints work:

```bash
cd backend
uv run uvicorn app.main:app --reload
```

Visit `http://localhost:8000/docs` to see the auto-generated OpenAPI docs for your new Projects endpoints.

### Step 9: Generate Frontend API Client

After adding the backend routes, regenerate the TypeScript API client:

```bash
cd frontend
npm run generate
```

This creates the API functions in `app/lib/api/gen/` that you'll use in the frontend.

### Step 10: Build Frontend Components

Create the projects list page at `frontend/app/routes/protected/projects.tsx`:

```tsx
import { useState, useEffect } from "react";
import { useNavigate } from "react-router";
import { listProjects, deleteProject } from "~/lib/api/gen/projects";
import type { ProjectResponse } from "~/lib/api/gen/model";

export default function ProjectsPage() {
  const navigate = useNavigate();
  const [projects, setProjects] = useState<ProjectResponse[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    loadProjects();
  }, []);

  async function loadProjects() {
    setLoading(true);
    try {
      const data = await listProjects();
      setProjects(data ?? []);
    } catch (err) {
      setError("Failed to load projects");
      console.error("Error loading projects:", err);
    } finally {
      setLoading(false);
    }
  }

  async function handleDelete(projectId: number) {
    if (!confirm("Are you sure you want to delete this project?")) return;
    try {
      await deleteProject(projectId);
      await loadProjects();
    } catch (err) {
      setError("Failed to delete project");
    }
  }

  if (loading) {
    return (
      <div className="flex justify-center p-8">
        <span className="loading loading-spinner loading-lg" />
      </div>
    );
  }

  return (
    <div className="container mx-auto p-6">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">Projects</h1>
        <button
          className="btn btn-primary"
          onClick={() => navigate("/projects/new")}
        >
          Create Project
        </button>
      </div>

      {error && (
        <div className="alert alert-error mb-4">
          <span>{error}</span>
        </div>
      )}

      {projects.length === 0 ? (
        <div className="text-center py-8">
          <p className="text-base-content/60 mb-4">No projects yet</p>
          <button
            className="btn btn-primary"
            onClick={() => navigate("/projects/new")}
          >
            Create Your First Project
          </button>
        </div>
      ) : (
        <div className="grid gap-4">
          {projects.map((project) => (
            <div key={project.id} className="card bg-base-100 shadow-xl">
              <div className="card-body">
                <h2 className="card-title">{project.name}</h2>
                {project.description && <p>{project.description}</p>}
                <p className="text-sm text-base-content/60">
                  Created: {new Date(project.created_at).toLocaleDateString()}
                </p>
                <div className="card-actions justify-end">
                  <button
                    className="btn btn-outline btn-sm"
                    onClick={() => navigate(`/projects/${project.id}`)}
                  >
                    View
                  </button>
                  <button
                    className="btn btn-outline btn-sm"
                    onClick={() => navigate(`/projects/${project.id}/edit`)}
                  >
                    Edit
                  </button>
                  <button
                    className="btn btn-error btn-outline btn-sm"
                    onClick={() => handleDelete(project.id)}
                  >
                    Delete
                  </button>
                </div>
              </div>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

Create the new project form at `frontend/app/routes/protected/projects-new.tsx`:

```tsx
import { useState } from "react";
import { useNavigate } from "react-router";
import { createProject } from "~/lib/api/gen/projects";

export default function NewProjectPage() {
  const navigate = useNavigate();
  const [name, setName] = useState("");
  const [description, setDescription] = useState("");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    if (!name.trim()) {
      setError("Project name is required");
      return;
    }
    setLoading(true);
    setError(null);
    try {
      await createProject({ name, description: description || undefined });
      navigate("/projects");
    } catch (err) {
      setError("Failed to create project");
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="container mx-auto p-6 max-w-md">
      <h1 className="text-3xl font-bold mb-6">Create New Project</h1>

      {error && (
        <div className="alert alert-error mb-4">
          <span>{error}</span>
        </div>
      )}

      <form onSubmit={handleSubmit} className="space-y-4">
        <div className="form-control">
          <label className="label" htmlFor="name">
            <span className="label-text">Project Name *</span>
          </label>
          <input
            id="name"
            type="text"
            className="input input-bordered"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
            disabled={loading}
          />
        </div>

        <div className="form-control">
          <label className="label" htmlFor="description">
            <span className="label-text">Description</span>
          </label>
          <textarea
            id="description"
            className="textarea textarea-bordered"
            rows={3}
            value={description}
            onChange={(e) => setDescription(e.target.value)}
            disabled={loading}
          />
        </div>

        <div className="flex gap-2">
          <button
            type="submit"
            className="btn btn-primary flex-1"
            disabled={loading}
          >
            {loading ? (
              <span className="loading loading-spinner loading-sm" />
            ) : (
              "Create Project"
            )}
          </button>
          <button
            type="button"
            className="btn btn-outline"
            onClick={() => navigate("/projects")}
            disabled={loading}
          >
            Cancel
          </button>
        </div>
      </form>
    </div>
  );
}
```

### Step 11: Register Routes

Update `frontend/app/routes.ts` to include the new project routes:

```typescript
import { type RouteConfig, layout, route } from "@react-router/dev/routes";

export default [
  layout("routes/protected/layout.tsx", [
    // ... existing routes ...
    route("projects", "routes/protected/projects.tsx"),
    route("projects/new", "routes/protected/projects-new.tsx"),
    route("projects/:id/edit", "routes/protected/projects-edit.tsx"),
  ]),
] satisfies RouteConfig;
```

### Step 12: Add Navigation

Update your sidebar component to include the projects link:

```tsx
<li>
  <NavLink to="/projects" className={({ isActive }) => isActive ? "active" : ""}>
    Projects
  </NavLink>
</li>
```

### Step 13: Test the Complete Feature

**1. Start the database** (if not already running):

```bash
docker compose up db -d
```

**2. Start the backend**:

```bash
cd backend
uv run uvicorn app.main:app --reload
```

**3. Start the frontend** (in a new terminal):

```bash
cd frontend
npm run dev
```

**4. Navigate to projects**: Visit `http://localhost:5173/projects`

**5. Test CRUD operations**: Create, view, edit, and delete projects.

### Summary

You've successfully added a complete "Projects" feature that demonstrates:

- **Database design**: Migration with proper foreign keys and constraints
- **Backend architecture**: Repository pattern, service layer, API routes
- **Dependency injection**: Proper container configuration
- **API design**: RESTful endpoints with proper HTTP status codes
- **Frontend integration**: API client generation and React components
- **Security**: Organization-based data isolation and authentication

This pattern can be applied to add any new entity to your FastReact application. The key principles are:

1. Start with the database schema
2. Build from the data layer up (repository → service → routes)
3. Configure dependency injection
4. Generate and use the API client on the frontend
5. Build React components following the existing design patterns
