## 1. Understanding the Core Concept: "Family Bank"

The "Family Bank" is essentially:

* **A Gamified Educational Financial System:** It uses game-like mechanics (earning rewards, paying costs) within the family unit to teach children fundamental financial principles in a controlled, safe environment.
* **Parent as Central Authority:** Parents act like a combination of a central bank and government. They have the authority to:
    * **"Mint" Currency:** Create the internal, symbolic currency without needing real-world backing or balancing (the "infinite sink" concept).
    * **Set Economic Rules:** Define the "value" of tasks (earning potential), the "cost" of privileges or goods/services, and implement system-wide rules like taxes, fees, penalties, or even simulated inflation.
    * **Govern Transactions:** Approve or deny certain actions (like chore completion payouts or large expenditures).
    * **Oversee the System:** Monitor activity through audit trails.
* **Children as Participants:** Children operate within this economy. They can:
    * **Earn:** Acquire currency by completing predefined tasks or chores.
    * **Spend:** Use their currency to access privileges, activities, or potentially tangible items (defined as "products" or "services" within the system).
    * **Deposit/Hold:** Their balance represents their accumulated wealth.
    * **Transfer (Future):** Exchange currency with siblings or potentially other family members (inter-family).
* **Focus on Concepts, Not Scarcity:** Unlike a real bank, the system doesn't need balanced books. Its primary goal is to model concepts like earning, spending, value, supply/demand (implicitly through pricing), consequences (penalties), and potentially more complex ideas like taxation and inflation.
* **Key Abstractions:**
    * **Currency:** The internal unit of exchange (e.g., "Family Dollars", "Points").
    * **Products/Services:** Anything currency can be exchanged for (e.g., screen time, a specific toy, choosing dinner, help with homework). These have associated *costs*.
    * **Tasks:** Actions children can take to earn currency. These have associated *rewards*.
    * **Transactions:** Records of currency movement (earning from tasks, spending on products/services, transfers, fees, taxes).
    * **Policies:** The rules set by the Parent/Banker (costs, rewards, tax rates, approval requirements).

## 2. MVP Definition (API-First)

To get started with dogfooding, let's define a minimal viable product focusing on the absolute core loop: **Earning via tasks and Spending on predefined items/privileges, all managed by a Parent.**

**Goal:** Enable a single family to track earnings from basic chores and spending on basic privileges with parent approval.

**User Roles (MVP):**

1.  **Parent (Banker):**
    * Can log in/authenticate.
    * Can create/view/update/delete `Child` accounts within their family.
    * Can define `Tasks` (name, description, reward amount).
    * Can define `Privileges` (name, description, cost amount).
    * Can view pending `Transaction` requests (task completion, privilege spending).
    * Can approve or reject pending `Transactions`.
    * Can manually adjust a `Child`'s balance (for initial setup or corrections).
    * Can view the transaction history (ledger) for each `Child`.
    * Can view the current balance for each `Child`.
2.  **Child (Accountholder):**
    * (Initially, actions might be logged *by the parent* via the API, or via a very simple interface later).
    * Conceptually, the Child performs a task or requests a privilege.
    * Can view their own balance.
    * Can view their own transaction history.

**Core Features (MVP):**

1.  **Family & User Management:**
    * Ability for a Parent to register/set up their "Family Bank".
    * Ability for a Parent to add/manage Child accounts linked to their family.
    * Basic authentication for the Parent role.
2.  **Task Management:**
    * Parent defines tasks and their currency reward value.
3.  **Privilege Management:**
    * Parent defines privileges/items and their currency cost.
4.  **Transaction Logging & Approval:**
    * Mechanism to log a request for task completion (creating a *pending* "earn" transaction).
    * Mechanism to log a request for privilege usage (creating a *pending* "spend" transaction).
    * Parent approves/rejects pending transactions, which then updates the child's balance and marks the transaction as completed/rejected.
    * Direct manual adjustments by the parent (logged as a specific transaction type, e.g., "adjustment").
5.  **Balance & Ledger Viewing:**
    * Ability to retrieve the current balance for any child.
    * Ability to retrieve a list of transactions for a child (their statement/ledger).

**Explicitly Excluded from MVP:**

* Dynamic pricing/inflation.
* Inter-sibling/Inter-family transfers or exchanges.
* Automated taxes or penalties.
* Appeals mechanism.
* Advanced reporting, analytics, gamification (leaderboards, etc.).
* Reward store complexities (inventory, etc.).
* Automated proof of task completion (QR codes, photos).
* Notifications.
* Child login/direct interaction (Parent handles logging initially via API).
* Frontend UI (API only).

## 3. Basic Backend Architecture (Clean Architecture - API First)

Applying the Clean Architecture structure to Family Bank MVP:

---

**`src/core/`**

* **`domain/`**: Plain data classes/objects.
    * `family.py`: `Family(id: UUID, name: str)`
    * `user.py`: `User(id: UUID, family_id: UUID, name: str, role: UserRole, balance: Decimal = Decimal(0))` (Balance relevant for Child role)
    * `common.py`: `UserRole(Enum)` -> `PARENT`, `CHILD`
    * `task.py`: `Task(id: UUID, family_id: UUID, name: str, description: str | None, reward: Decimal)`
    * `privilege.py`: `Privilege(id: UUID, family_id: UUID, name: str, description: str | None, cost: Decimal)`
    * `transaction.py`: `Transaction(id: UUID, user_id: UUID, family_id: UUID, type: TransactionType, amount: Decimal, status: TransactionStatus, description: str | None, related_task_id: UUID | None, related_privilege_id: UUID | None, timestamp: datetime, approved_by: UUID | None, approved_at: datetime | None)`
    * `common.py`: `TransactionType(Enum)` -> `EARN`, `SPEND`, `ADJUSTMENT`, `PENALTY` (Add later) `TransactionStatus(Enum)` -> `PENDING`, `APPROVED`, `REJECTED`

* **`repositories/`**: Abstract Base Classes (ABCs) for data access.
    * `family_repo.py`: `FamilyRepository(ABC)` (methods: `save`, `get_by_id`, `find_by_name`)
    * `user_repo.py`: `UserRepository(ABC)` (methods: `save`, `get_by_id`, `list_by_family`, `get_balance`, `update_balance`)
    * `task_repo.py`: `TaskRepository(ABC)` (methods: `save`, `get_by_id`, `list_by_family`, `delete`)
    * `privilege_repo.py`: `PrivilegeRepository(ABC)` (methods: `save`, `get_by_id`, `list_by_family`, `delete`)
    * `transaction_repo.py`: `TransactionRepository(ABC)` (methods: `save`, `get_by_id`, `list_by_user`, `list_pending_by_family`, `update_status`)

* **`use_cases/`**: DTOs (Pydantic models) for API boundaries and potentially use case interfaces (though often implementation suffices).
    * `family_dtos.py`: `FamilyCreateDTO`, `FamilyReadDTO`
    * `user_dtos.py`: `UserCreateDTO` (for Child), `UserReadDTO`, `BalanceReadDTO`
    * `task_dtos.py`: `TaskCreateDTO`, `TaskUpdateDTO`, `TaskReadDTO`
    * `privilege_dtos.py`: `PrivilegeCreateDTO`, `PrivilegeUpdateDTO`, `PrivilegeReadDTO`
    * `transaction_dtos.py`: `TransactionCreateDTO` (specific for log_task, log_spend, manual_adjust), `TransactionReadDTO`, `TransactionApprovalDTO` (status, approver_id)

* **`services/`**: ABCs for external services.
    * `auth_service.py`: `AuthService(ABC)` (methods: `create_token`, `verify_token`, `get_user_id_from_token`)

---

**`src/infrastructure/`**

* **`persistence/sqlalchemy/`**: SQLAlchemy implementations.
    * `models.py`: Define SQLAlchemy tables (`families`, `users`, `tasks`, `privileges`, `transactions`) mapping to domain entities. Use `Numeric` for currency. Include relationships (e.g., User to Family, Transaction to User).
    * `family_repo.py`, `user_repo.py`, `task_repo.py`, `privilege_repo.py`, `transaction_repo.py`: Concrete implementations of the repository ABCs using SQLAlchemy session/queries. Handle balance updates within transactions carefully (atomic updates).
* **`services/jwt/`**: JWT implementation.
    * `auth_service.py`: Implement `AuthService` using a library like `python-jose`.

---

**`src/application/`**

* **`use_cases/`**: Concrete use case implementations. Inject repository and service *interfaces*.
    * `family_uc.py`: `RegisterFamilyUseCase`, `GetFamilyUseCase`.
    * `user_uc.py`: `AddChildUseCase`, `ListChildrenUseCase`, `GetUserBalanceUseCase`, `ManualAdjustBalanceUseCase`.
    * `task_uc.py`: `DefineTaskUseCase`, `ListTasksUseCase`, `UpdateTaskUseCase`, `DeleteTaskUseCase`.
    * `privilege_uc.py`: `DefinePrivilegeUseCase`, `ListPrivilegesUseCase`, `UpdatePrivilegeUseCase`, `DeletePrivilegeUseCase`.
    * `transaction_uc.py`: `LogTaskCompletionUseCase` (creates PENDING earn TX), `LogPrivilegeUsageUseCase` (creates PENDING spend TX), `ApproveRejectTransactionUseCase` (updates TX status, updates user balance if approved), `ListTransactionsUseCase`, `ListPendingTransactionsUseCase`.

---

**`src/api/`**

* **`main.py`**: FastAPI app setup. Database initialization (SQLAlchemy engine, session factory). Dependency Injection wiring (provide concrete repo/service implementations). Include API routers.
* **`routes/`**: FastAPI routers using DTOs. Inject and call use cases.
    * `auth_routes.py`: `/token` (login for Parent).
    * `family_routes.py`: `/families` (POST for registration, maybe GET later).
    * `user_routes.py`: `/families/{family_id}/users` (POST for child), `/families/{family_id}/users` (GET), `/users/{user_id}/balance` (GET). `/users/{user_id}/adjust_balance` (POST, Parent only).
    * `task_routes.py`: `/families/{family_id}/tasks` (POST, GET), `/tasks/{task_id}` (PUT, DELETE).
    * `privilege_routes.py`: `/families/{family_id}/privileges` (POST, GET), `/privileges/{privilege_id}` (PUT, DELETE).
    * `transaction_routes.py`: `/transactions/log-task` (POST), `/transactions/log-privilege` (POST), `/families/{family_id}/transactions/pending` (GET), `/transactions/{transaction_id}/approve` (POST), `/transactions/{transaction_id}/reject` (POST), `/users/{user_id}/transactions` (GET).
* **`middleware/`**:
    * `auth.py`: Middleware to verify JWT token (using `AuthService`) and potentially attach authenticated user data to the request state. Enforce Parent role for specific endpoints.
* **`dependencies.py`**: FastAPI dependency functions (e.g., `get_db_session`, `get_current_parent_user` based on token).

---

**`src/config.py`**: Load settings (DB URL, JWT secret, etc.) from environment variables (`.env`) using Pydantic's `BaseSettings`.

---
