# Centeva Packages
Centeva packages is the backend architecture we use in most apps. The main idea is for services to inherit common implementations for data access.

## Data Structure

![png](image1.png

## Application Layer

### Client
The client is the front-end app. It will make API Request using a `DataService`. These API requests map to `Controllers`.
### Controller
`Controllers` define API endpoints and their types. 90% of CRUD operations can be done with the built-in service methods but you can create your own. **Controllers should not have any logic!**

#### Example Controller
```cs
using System.Collections.Generic;
using System.Web.Http;
using Example.BLL;
using Example.BLL.Services;
using Example.DAL.Models;

namespace Example.Client.Controllers {
	public class TestController:ApiController {
		private readonly TestService _service;

		public TestController(ExampleServiceProvider service) {
			_service = service.TestService;
		}

		[HttpGet]
		public Test Read(int id) {
			return _service.Read(x => x.Id == id);
		}

		[HttpGet]
		public IEnumerable<Test> ReadList()
		{
			return _service.ReadList();
		}

		[HttpPost]
		public Test Create(Test model) {
			return _service.Create(model);
		}

		[HttpPost]
		public IEnumerable<Test> Create(List<Test> models) {
			return _service.Create(models);
		}

		[HttpPost]
		public Test Update(Test model) {
			return _service.Update(model);
		}

		[HttpPost]
		public IEnumerable<Test> Update(List<Test> models) {
			return _service.Update(models);
		}

		[HttpPost]
		public int DeleteById(int id) {
			return _service.Delete(x => x.Id == id);
		}
	}
}
```
## BLL / Service Layer

### Services
`Services` hold all of our backend logic. `Services` can be called from `Controllers` or other `Services`.
*Note: Older versions of of our architecture(Inspections, Licensing, OL) will have an interface on every service to make testing easier, This is redundant on new projects.*
#### Example Service
```cs
using Centeva.Interfaces.DataAccess;
using Example.DAL.Models;

namespace Example.BLL.Services {
	public class TestService:ExampleService<Test> {
		public TestService(IDataAccess<Test> dataAccess, ExampleServiceProvider serviceProvider) :
			base(dataAccess, serviceProvider) {}
	}
}
```
### DTOs
`Data Transfer Objects` are normally what we pass to a `controller`. They can have computed fields that are not stored in the database. `Services` will map `Entities` to `DTOs` or will pass back just an `Entity`. *Note: Some people think we should avoid passing Entities over the wire.*

One pattern we have started using with `DTOs` is to add mapping expressions to them. This consolidates mapping logic and work nicely with LINQ.
#### Example DTO
```cs
using System;
using System.Linq.Expressions;
using Example.DAL.Models;

namespace Example.BLL.Dto {
	public class TestDetails {
		public int Id { get; set; }
		public string Name { get; set; }
		public string Number { get; set; }
		public string TypeCode { get; set; }
		public string CategoryCode { get; set; }
		public int? SiteId { get; set; }

		public static readonly Expression<Func<Test, TestDetails>> FromTest = t => new TestDetails
		{
			Id = t.OtherId,
			CategoryCode = t.Other.OtherCategoryTypeCode,
			TypeCode = t.Other.OtherTypeCode,
			Name = t.Other.Name,
			Number = t.Other.OtherNum,
			SiteId = t.Other.SiteId
		};
	}
}
```

## Data Layer

### Generic Data Access

All of our services inherit from a `Generic Data Access` from CentevaPackages. 
#### Default Operations
- Read
- ReadList
- Count
- Create
- Update
- Delete

Each Operation has four lifecycle hooks.
#### Lifecycle Hooks
- Verify
- Before
- Action
- After

So the Create action lifecycle hooks would look like this.
```
VerifyCreateAuthorization
BeforeCreateData
CreateData
AfterCreateData
```

We can implement these hooks in our `services`.
#### Example BeforeUpdate hook
```cs
protected override List<Test> BeforeUpdateData(List<Test> models) {
	models.ForEach(x => {
		x.Coding = null;
		x.TestDepartmentType = null;
		x.TestLevelType = null;
	});
	return models;
}
```

### Models / Entities
We use Entity Framework with Code First `Migrations`, which means our database schema is driven by C# classes. We call those classes `Entities` because they corespond to tables in the database through Entity Framework. Any changes to these `Entities` require a `Migration`.

### Migrations
Entity Framework has the idea of Code First `Migrations`, these are how `Model` changes are applied to the database. In EF5 `Migrations` have to be applied sequencially. *note: Only one person can make a migration at a time! Most teams have a migration item to help with this.*