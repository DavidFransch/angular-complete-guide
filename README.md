# Angular Complete Guide 2023

## Versioning
(this course uses latest)
Angular 1 = AngularJS
Rest of versions = Angular

## Property & Event Binding
- HTML Elements = Native properties & events
- Directives = Custom properties & events
- Components = Custom properties & events

## Binding to custom properties

See `app-component.ts`:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  serverElements = [{type: "server", name: "Test server", content: "Some test content"}];
}
```

In `app-component.html` (i.e. parent of `server-element.component.html`) we can bind the serverElement to element with `[element]="serverElement"`:

```angular2html
<div class="container">
  <app-cockpit></app-cockpit>
  <hr>
  <div class="row">
    <div class="col-xs-12">
      <app-server-element
        *ngFor="let serverElement of serverElements"
        [element]="serverElement"
      >

      </app-server-element>
    </div>
  </div>
</div>

```

In `server-element.component.ts` we need to use the `@Input()` decorator:

```typescript
import {Component, Input, OnInit} from '@angular/core';

@Component({
  selector: 'app-server-element',
  templateUrl: './server-element.component.html',
  styleUrls: ['./server-element.component.css']
})
export class ServerElementComponent implements OnInit {

  @Input() element: {type: string, name: string, content: string};

  constructor() { }

  ngOnInit(): void {
  }

}

```

This allows us to access the element data provided by serverElements in `server-element.component.html`:

```angular2html
<div
  class="panel panel-default">
  <div class="panel-heading">{{ element.name }}</div>
  <div class="panel-body">
    <p>
      <strong *ngIf="element.type === 'server'" style="color: red">{{ element.content }}</strong>
      <em *ngIf="element.type === 'blueprint'">{{ element.content }}</em>
    </p>
  </div>
</div>
```

## Binding to custom events

In `cockpit.component.html`:
```angular2html
<div class="row">
  <div class="col-xs-12">
    <p>Add new Servers or blueprints!</p>
    <label>Server Name</label>
<!--    <input type="text" class="form-control" [(ngModel)]="newServerName">-->
    <input
      type="text"
      class="form-control"
      #serverNameInput
    >
    <label>Server Content</label>
<!--    <input type="text" class="form-control" [(ngModel)]="newServerContent">-->
    <input
      type="text"
      class="form-control"
      #serverContentInput
    >
    <br>
    <button
      class="btn btn-primary"
      (click)="onAddServer(serverNameInput)">Add Server</button>
    <button
      class="btn btn-primary"
      (click)="onAddBlueprint(serverNameInput)">Add Server Blueprint</button>
  </div>
</div>
```

Using EventEmitter to emit events to parent component, see child component `cockpit.components.ts`:

```typescript
import {Component, EventEmitter, OnInit, Output} from '@angular/core';

@Component({
  selector: 'app-cockpit',
  templateUrl: './cockpit.component.html',
  styleUrls: ['./cockpit.component.css']
})
export class CockpitComponent implements OnInit {

  @Output() serverCreated = new EventEmitter<{serverName: string, serverContent: string}>();
  @Output() blueprintCreated = new EventEmitter<{serverName: string, serverContent: string}>();
  newServerName = '';
  newServerContent = '';

  constructor() { }

  onAddServer() {
    this.serverCreated.emit({
      serverName: this.newServerName,
      serverContent: this.newServerContent
    });
  }

  onAddBlueprint() {
    this.blueprintCreated.emit({
      serverName: this.newServerName,
      serverContent: this.newServerContent
    });
  }

  ngOnInit(): void {
  }

}
```

Events are then emitted to `app.component.html`:

```angular2html
<div class="container">
  <app-cockpit
    (serverCreated)="onServerAdded($event)"
    (blueprintCreated)="onBlueprintAdded($event)"
    >
  </app-cockpit>
  <hr>
  <div class="row">
    <div class="col-xs-12">
      <app-server-element
        *ngFor="let serverElement of serverElements"
        [element]="serverElement"
      >
      </app-server-element>
    </div>
  </div>
</div>
```

Triggering functions in `app.component.ts`:
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  serverElements = [{type: "server", name: "Test server", content: "Some test content"}];

  onServerAdded(serverData: {serverName: string, serverContent: string}) {
    this.serverElements.push({
      type: 'server',
      name: serverData.serverName,
      content: serverData.serverContent
    });
  }

  onBlueprintAdded(blueprintData: {serverName: string, serverContent: string}) {
    this.serverElements.push({
      type: 'blueprint',
      name: blueprintData.serverName,
      content: blueprintData.serverContent
    });
  }
}

```

## Local References
- Access elements in template (ONLY)

In `cockpit.component.html`:
```java
<div class="row">
  <div class="col-xs-12">
    <p>Add new Servers or blueprints!</p>
    <input
      type="text"
      class="form-control"
      #serverNameInput
    >
    <label>Server Content</label>
    <input type="text" class="form-control" [(ngModel)]="newServerContent">
    <br>
    <button
      class="btn btn-primary"
      (click)="onAddServer(serverNameInput)">
        Add Server
      </button>
  </div>
</div>
```

In `cockpit.component.html`:

```typescript
  onAddServer(serverNameInput) {
    console.log(serverNameInput);
  }
```

Output looks like (we can also access properties like `serverNameInput.value`):
```bash
    <input
    _ngcontent-ady-c44
      type="text"
      class="form-control"
    >
```

## Getting Access to the Template & Dom with @ViewChild
(**use when we want to get access to an element before we call the method**)

In `cockpit.component.html`:

```angular2html
<div class="row">
  <div class="col-xs-12">
    <input
      type="text"
      class="form-control"
      #serverContentInput
    >
    ...
  </div>
</div>
```

```typescript
import {Component, ElementRef, EventEmitter, OnInit, Output, ViewChild} from '@angular/core';

@Component({
  selector: 'app-cockpit',
  templateUrl: './cockpit.component.html',
  styleUrls: ['./cockpit.component.css']
})
export class CockpitComponent implements OnInit {


  @ViewChild('serverContentInput', {static: true}) serverContentInput: ElementRef;

  constructor() { }

  onAddServer(serverNameInput: HTMLInputElement) {
    this.serverCreated.emit({
      serverName: serverNameInput.value,
      serverContent: this.serverContentInput.nativeElement.value
    });
  }

  ngOnInit(): void {
  }

}

```

## Projecting content into a component with ng-content

In `server-element.component.html`:
```angular2html
<div
  class="panel panel-default">
  <div class="panel-heading">{{ element.name }}</div>
  <div class="panel-body">
    <ng-content></ng-content>
  </div>
</div>
```

Then in  `app.component.html`:
```angular2html
<div class="container">
  <app-cockpit
    (serverCreated)="onServerAdded($event)"
    (blueprintCreated)="onBlueprintAdded($event)"
    >
  </app-cockpit>
  <hr>
  <div class="row">
    <div class="col-xs-12">
      <app-server-element
        *ngFor="let serverElement of serverElements"
        [element]="serverElement"
      >
          <p>
            <strong *ngIf="serverElement.type === 'server'" style="color: red">{{ serverElement.content }}</strong>
            <em *ngIf="serverElement.type === 'blueprint'">{{ serverElement.content }}</em>
          </p>
      </app-server-element>
    </div>
  </div>
</div>
```

## Understanding the component lifecycle
- `ngOnChanges`: called after a bound input changes
-  `ngOnInit`: called once the component is initialized
- `ngDoCheck`: called during every change detection run (called often i.e. event events/timers etc.)
    - e.g. of when to use = manually inform angular about some change that it would not be able to detect
- `ngAfterContentInit`: called after content (ng-content) has been projected into view
- `ngAfterContentChecked`: called every time the projected content has been checked
- `ngAfterViewInit`: called after the component's view and child views have been initialized
- `ngAfterViewChecked`: called every time the view (and child views) have been checked
- `ngOnDestroy`: called once the component us about to be destroyed

## Adding navigation with Event Binding and ngIf

In the course project `header.component.html`
we added `(click)` events:
```angular2html
<nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="navbar-header">
      <a href="#" class="navbar-brand">Recipe Book</a>
    </div>

    <div class="collapse navbar-collapse">
      <ul class="nav navbar-nav">
        <li><a href="#" (click)="onSelect('recipe')">Recipes</a></li>
        <li><a href="#" (click)="onSelect('shopping-list')">Shopping List</a></li>
      </ul>
      <ul class="nav navbar-nav navbar-right">
        <li class="dropdown">
          <a href="#" class="dropdown-toggle" role="button">Manage <span class="caret"></span></a>
          <ul class="dropdown-menu">
            <li><a href="#">Save Data</a></li>
            <li><a href="#">Fetch Data</a></li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</nav>
```

Implementing the `onSelect` method in `header.component.ts` and using EventEmitter to emit an event that can be used in the parent `app.component.html`:

```typescript
import {Component, EventEmitter, Output} from '@angular/core';

@Component({
  selector: 'app-header',
  templateUrl: './header.component.html'
})
export class HeaderComponent {
  @Output() featureSelected = new EventEmitter<string>();
  onSelect(feature: string) {
    this.featureSelected.emit(feature);
  }
}

```

Using the `featureSelected` property defined in `header.component.html`:

```angular2html
<app-header (featureSelected)="onNavigate($event)"></app-header>
<div class="container">
  <div class="row">
    <div class="col-md-12">
      <app-recipes *ngIf="loadedFeature === 'recipe'"></app-recipes>
      <app-shopping-list *ngIf="loadedFeature !== 'recipe'"></app-shopping-list>
    </div>
  </div>
</div>
```
We can then call `onNavigate($event)` and implement it in `app.component.ts`:

```typescript
<app-header (featureSelected)="onNavigate($event)"></app-header>
<div class="container">
  <div class="row">
    <div class="col-md-12">
      <app-recipes *ngIf="loadedFeature === 'recipe'"></app-recipes>
      <app-shopping-list *ngIf="loadedFeature !== 'recipe'"></app-shopping-list>
    </div>
  </div>
</div>
```

## Using Local References to add ingredients to shopping list

In `shopping-edit.component.html` we add references `#nameInput` and `#amountInput` as well as a click event on the add button:
```angular2html
<div class="row">
  <div class="col-xs-12">
    <form>
      <div class="row">
        <div class="col-sm-5 form-group">
          <label for="name">Name</label>
          <input type="text" id="name" class="form-control" #nameInput>
        </div>
        <div class="col-sm-2 form-group">
          <label for="amount">Amount</label>
          <input type="number" id="amount" class="form-control" #amountInput>
        </div>
      </div>
      <div class="row">
        <div class="col-xs-12">
          <button class="btn btn-success" type="submit" (click)="onAddItem()">Add</button>
          <button class="btn btn-danger" type="button">Delete</button>
          <button class="btn btn-primary" type="button">Clear</button>
        </div>
      </div>
    </form>
  </div>
</div>
```

In `shopping-edit.components.ts` we use `@ViewChild` for the local references and `@Output` to emit an event that can be used in the parent component shopping list:
```typescript
@Component({
  selector: 'app-shopping-edit',
  templateUrl: './shopping-edit.component.html',
  styleUrls: ['./shopping-edit.component.css']
})
export class ShoppingEditComponent implements OnInit {

  @ViewChild('nameInput', {static: false}) nameInputRef: ElementRef;
  @ViewChild('amountInput', {static: false}) amountInputRef: ElementRef;
  @Output() ingredientAdded = new EventEmitter<Ingredient>();

  constructor() {
  }

  ngOnInit() {
  }

  onAddItem() {
    const ingredientName = this.nameInputRef.nativeElement.value;
    const ingredientAmount = this.amountInputRef.nativeElement.value;
    const newIngredient = new Ingredient(ingredientName, ingredientAmount);
    this.ingredientAdded.emit(newIngredient);
  }
}
```

In `shopping-list.component.html`:

```angular2html
<div class="row">
  <div class="col-xs-10">
    <app-shopping-edit
      (ingredientAdded)="onIngredientAdded($event)"
    ></app-shopping-edit>
    <hr>
    <ul class="list-group">
      <a
        class="list-group-item"
        style="cursor: pointer"
        *ngFor="let ingredient of ingredients"
      >
        {{ ingredient.name }} ({{ ingredient.amount }})
      </a>
    </ul>
  </div>
</div>

```

Finally in `shopping-list.components.ts` we can implement `onIngredientAdded`:

```typescript
import { Ingredient } from '../shared/ingredient.model';

@Component({
  selector: 'app-shopping-list',
  templateUrl: './shopping-list.component.html',
  styleUrls: ['./shopping-list.component.css']
})
export class ShoppingListComponent implements OnInit {
  ingredients: Ingredient[] = [
    new Ingredient('Apples', 5),
    new Ingredient('Tomatoes', 10),
  ];
  
  onIngredientAdded(ingredient: Ingredient) {
    this.ingredients.push(ingredient);
  }
}
```
## Directives Deep Dive
### Attribute vs Structural

Attribute:
- Like normal HTML (possibly with data binding or event binding)
- Only affect/change element they are added to
  Structural Directives:
- Like HTML but have leading `*`
- Affect a whole area in the DOM (elements get added/removed)


### Creating an Attribute Directive

```typescript
import {Directive, ElementRef, OnInit, Renderer2} from "@angular/core";

@Directive({
  selector: '[appBetterHighlight]'
})

export class BetterHighlightDirective implements OnInit{
  constructor(private elRef: ElementRef, private renderer: Renderer2) {}

  ngOnInit() {
    this.renderer.setStyle(this.elRef.nativeElement, 'background-color', 'blue');
  }
}
```

```angular2html
<p appBetterHighlight>Style with a better directive!</p>
```

### Using:
#### HostListener to Host Events
#### Using HostBinding to Bind to Host Properties
#### Binding to Directive Properties (allowing users to select colors)

```typescript
import {Directive, ElementRef, HostBinding, HostListener, Input, OnInit, Renderer2} from "@angular/core";

@Directive({
  selector: '[appBetterHighlight]'
})

export class BetterHighlightDirective implements OnInit {
  @Input() defaultColor: string = 'transparent';
  @Input() highlightColor: string = 'blue';
  @HostBinding('style.backgroundColor') backgroundColor: string = this.defaultColor;

  constructor(private elRef: ElementRef, private renderer: Renderer2) {}

  ngOnInit() {
    this.backgroundColor = this.defaultColor;
    this.renderer.setStyle(this.elRef.nativeElement, 'background-color', 'blue');
  }

  @HostListener('mouseenter') mouseover(eventData: Event) {
    // this.renderer.setStyle(this.elRef.nativeElement, 'background-color', 'blue');
    this.backgroundColor = this.highlightColor;
  }

  @HostListener('mouseleave') mouseleave(eventData: Event) {
    // this.renderer.setStyle(this.elRef.nativeElement, 'background-color', 'transparent');
    this.backgroundColor = this.defaultColor;
  }
}
```

```angular2html
<p appBetterHighlight [defaultColor]="'yellow'" [highlightColor]="'red'">Style with a better directive!</p>
```

### What happens behind the scenes on structural directives

This standard syntax is converted to the `ng-template` shown below:
```angular2html
    <span *ngIf="onlyOdd">
            <p>Only odd</p>
    </span>
```
See `ng-template`:
```angular2html
    <ng-template [ngIf]="onlyOdd">
        <span>
          <p>Only odd</p>
        </span>
    </ng-template>
```

### Understanding ngSwitch

```angular2html
<div [ngSwitch]="value">
  <p *ngSwitchCase="5">Value is 5</p>
  <p *ngSwitchCase="10">Value is 10</p>
  <p *ngSwitchCase="100">Value is 100</p>
  <p *ngSwitchDefault>Value is Default</p>
</div>

```

### Creating a dropdown using directives

```typescript
import {Directive, ElementRef, HostBinding, HostListener} from '@angular/core';
 
@Directive({
  selector: '[appDropdown]'
})
export class DropdownDirective {
  @HostBinding('class.open') isOpen = false;
  @HostListener('document:click', ['$event']) toggleOpen(event: Event) {
    this.isOpen = this.elRef.nativeElement.contains(event.target) ? !this.isOpen : false;
  }
  constructor(private elRef: ElementRef) {}
}
```

```angular2html
<ul class="nav navbar-nav navbar-right">
  <li class="dropdown" appDropdown>
    <a href="#" class="dropdown-toggle" role="button">Manage <span class="caret"></span></a>
    <ul class="dropdown-menu">
      <li><a href="#">Save Data</a></li>
      <li><a href="#">Fetch Data</a></li>
    </ul>
  </li>
</ul>
```

## Services and Dependency Injection
(Code can be found in `angular-udemy-course/`)
### Hierarchical Injector

#### Creating a Logging and Data Service
For the logging service we create a `logging.service.ts`:

```typescript
export class LoggingService {
  logStatusChange(status: string) {
    console.log('A server status changed, new status: ' + status);
  }
}
```

For the data service we move our static array and methods into a new service file called,
`accounts.service.ts` (which we extracted from `app.component.ts`):

```typescript
export class AccountsService {
  accounts = [
    {
      name: 'Master Account',
      status: 'active'
    },
    {
      name: 'Testaccount',
      status: 'inactive'
    },
    {
      name: 'Hidden Account',
      status: 'unknown'
    }
  ];

  addAccount(name: string, status: string) {
    this.accounts.push({name: name, status: status})
  }

  updateStatus(id: number, status: string) {
    this.accounts[id].status = status;
  }
}
```

In the `account.component.ts` we use the service by creating a constructor and adding the service as a provider in the providers array:\
(NOTE: if we add the AccountService to the providers array we create a new instance of it)
```typescript
import { Component, Input } from '@angular/core';
import {LoggingService} from "../logging.service";

@Component({
  selector: 'app-account',
  templateUrl: './account.component.html',
  styleUrls: ['./account.component.css'],
  providers: [LoggingService]
})
export class AccountComponent {
  @Input() account: {name: string, status: string};
  @Input() id: number;

  constructor(private loggingService: LoggingService,
              private accountsService: AccountsService) {}

  onSetTo(status: string) {
    this.accountsService.updateStatus(this.id, status);
    this.loggingService.logStatusChange(status);
  }
}
```

In the `new-account.component.ts` we have:
```typescript
import { Component, EventEmitter, Output } from '@angular/core';
import { LoggingService } from "../logging.service";
import {AccountsService} from "../accounts.service";

@Component({
  selector: 'app-new-account',
  templateUrl: './new-account.component.html',
  styleUrls: ['./new-account.component.css'],
  providers: [LoggingService]
})
export class NewAccountComponent {
  // Note: We no longer need this
  // @Output() accountAdded = new EventEmitter<{name: string, status: string}>();

  constructor(private loggingService: LoggingService, private accountsService: AccountsService) {}

  onCreateAccount(accountName: string, accountStatus: string) {
    this.accountsService.addAccount(accountName, accountStatus);
    this.loggingService.logStatusChange(accountStatus);

    // Note: no longer need this
    // this.accountAdded.emit({
    //   name: accountName,
    //   status: accountStatus
    // });

    // Note: We shouldn't do this (instead we use dependency injection)
    // const service = new LoggingService();
    // service.logStatusChange(accountStatus);
  }
}
```

Finally, our `app.component.html` looks like this:
```angular2html
<div class="container">
  <div class="row">
    <div class="col-xs-12 col-md-8 col-md-offset-2">
      <app-new-account></app-new-account>
      <hr>
      <app-account
        *ngFor="let acc of accounts; let i = index"
        [account]="acc"
        [id]="i"
        ></app-account>
    </div>
  </div>
</div>

```
### Injecting Services into Services
We first add the services into `app.module.ts` like this
`providers: [AccountsService, LoggingService]`:

Next we can remove the logging service from `new-account.component.ts` and
`account.component.ts` and add it to our `accounts.service.ts`:\
(Note: we need to add `Injectable()` since we are injecting the service into another service)

```typescript
import {Injectable} from "@angular/core";

@Injectable()
export class AccountsService {
  accounts = [
    {
      name: 'Master Account',
      status: 'active'
    },
    {
      name: 'Testaccount',
      status: 'inactive'
    },
    {
      name: 'Hidden Account',
      status: 'unknown'
    }
  ];

  constructor(private loggingService: LoggingService) {}

  addAccount(name: string, status: string) {
    this.accounts.push({name: name, status: status})
    this.loggingService.logStatusChange(status);
  }

  updateStatus(id: number, status: string) {
    this.accounts[id].status = status;
    this.loggingService.logStatusChange(status);
  }
}
```
### Using Services for Cross-Component Communication

Add an event emitter in the `accounts.service.ts`:\
`statusUpdated = new EventEmitter<string>();`

Then in `account.component.ts` we emit the status in the update status method:\
`this.accountsService.statusUpdated.emit(status);`

Lastly, in the `new-account.component.ts` we can subscribe to the emitted status:
```typescript
export class NewAccountComponent {

  constructor(private loggingService: LoggingService, private accountsService: AccountsService) {
    this.accountsService.statusUpdated.subscribe(
      (status: string) => alert('New Status '+  status)
    );
  }

  onCreateAccount(accountName: string, accountStatus: string) {
    this.accountsService.addAccount(accountName, accountStatus);
  }
}
```

### Different way of injecting services
- If you're using Angular 6+ (check your package.json  to find out), you can provide application-wide services in a different way.

- Instead of adding a service class to the` providers[]`  array in `AppModule`, you can set the following config in `@Injectable()` :

`@Injectable({providedIn: 'root'})`\
`export class MyService { ... }`\
This is exactly the same as:

`export class MyService { ... }`\
and

```typescript
import { MyService } from './path/to/my.service';

@NgModule({
...
providers: [MyService]
})

export class AppModule { ... }
```
Using this syntax is completely optional, the traditional syntax (using providers[] ) will also work.

The "new syntax" does offer one advantage though: Services can be loaded lazily by Angular (behind the scenes) and redundant code can be removed automatically. This can lead to a better performance and loading speed - though this really only kicks in for bigger services and apps in general.

## Routing

### Registering some routes
In `app.module.ts`:
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';


import { AppComponent } from './app.component';
import { HomeComponent } from './home/home.component';
import { UsersComponent } from './users/users.component';
import { ServersComponent } from './servers/servers.component';
import { UserComponent } from './users/user/user.component';
import { EditServerComponent } from './servers/edit-server/edit-server.component';
import { ServerComponent } from './servers/server/server.component';
import { ServersService } from './servers/servers.service';
import {RouterModule, Routes} from "@angular/router";

// Create routes
const appRoutes: Routes = [
  { path: '', component:  HomeComponent },
  { path: 'users', component:  UsersComponent },
  { path: 'servers', component:  ServersComponent },
];

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent,
    UsersComponent,
    ServersComponent,
    UserComponent,
    EditServerComponent,
    ServerComponent
  ],
  
  // Register routes
  imports: [
    BrowserModule,
    FormsModule,
    RouterModule.forRoot(appRoutes),
  ],
  providers: [ServersService],
  bootstrap: [AppComponent]
})
export class AppModule { }

```

Creating a routing module with routes and child routes:
```typescript
import {RouterModule, Routes} from "@angular/router";
import {RecipesComponent} from "./recipes/recipes.component";
import {ShoppingListComponent} from "./shopping-list/shopping-list.component";
import {NgModule} from "@angular/core";
import {RecipeStartComponent} from "./recipes/recipe-start/recipe-start.component";
import {RecipeDetailComponent} from "./recipes/recipe-detail/recipe-detail.component";
import {RecipeEditComponent} from "./recipes/recipe-edit/recipe-edit.component";

  const appRoutes: Routes = [
    { path: '', redirectTo: '/recipes', pathMatch: 'full' },
    { path: 'recipes', component: RecipesComponent, children: [
        { path: '', component: RecipeStartComponent },
        { path: 'new', component: RecipeEditComponent },
        { path: ':id', component: RecipeDetailComponent },
        { path: ':id/edit', component: RecipeEditComponent }
      ]},
    { path: 'shopping-list', component: ShoppingListComponent }
  ]
@NgModule({
  imports: [RouterModule.forRoot(appRoutes)],
  exports: [RouterModule]
})
export class AppRoutingModule {
}

```

In `app.component.html` we use the header compoennt:
```angular2html
<app-header></app-header>
<div class="container">
  <div class="row">
    <div class="col-md-12">
      <router-outlet></router-outlet>
    </div>
  </div>
</div>

```

The links or navigation are in the header:
(`routerLinkActive="active"`) is used to dynamically set the link to active when on that route.
```angular2html
<nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="navbar-header">
      <a href="#" class="navbar-brand">Recipe Book</a>
    </div>

    <div class="collapse navbar-collapse">
      <ul class="nav navbar-nav">
        <li routerLinkActive="active"><a routerLink="/recipes">Recipes</a></li>
        <li routerLinkActive="active"><a routerLink="/shopping-list">Shopping List</a></li>
      </ul>
      <ul class="nav navbar-nav navbar-right">
        <li class="dropdown" appDropdown>
          <a style="cursor: pointer" class="dropdown-toggle" role="button">Manage <span class="caret"></span></a>
          <ul class="dropdown-menu">
            <li><a>Save Data</a></li>
            <li><a>Fetch Data</a></li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</nav>

```

## Observables
Creating a basic observable:\
(Unsubscribing prevents memory leaks but is only necessary for custom observables)
```typescript
import {Component, OnDestroy, OnInit} from '@angular/core';

import {interval, Subscription} from "rxjs";

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit, OnDestroy {

  private firstObsSubscription: Subscription;

  constructor() { }

  ngOnInit() {
    this.firstObsSubscription = interval(1000).subscribe((count) => {
      console.log(count);
    })
  }

  ngOnDestroy() {
    this.firstObsSubscription.unsubscribe();
  }

}
```

### Building a custom observable:

```typescript
import {Component, OnDestroy, OnInit} from '@angular/core';
import {interval, Observable, Subscription} from "rxjs";

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})

export class HomeComponent implements OnInit, OnDestroy {

  private firstObsSubscription: Subscription;

  constructor() { }

  ngOnInit() {
    const customIntervalObservable = Observable.create(observer => {
      let count = 0;
      setInterval(() => {
        observer.next(count);
        count++;
      }, 1000)
    });
    this.firstObsSubscription = customIntervalObservable.subscribe(data => {
      console.log(data);
    });
  }

  ngOnDestroy() {
    this.firstObsSubscription.unsubscribe();
  }
}
```

### Errors and completion:
```typescript
  ngOnInit() {
    const customIntervalObservable = Observable.create(observer => {
      let count = 0;
      setInterval(() => {
        observer.next(count);
        if(count === 2){
          observer.complete();
        }
        if(count > 3) {
          observer.error(new Error('Count is greater 3!!!'))
        }
        count++;
      }, 1000)
    });
    this.firstObsSubscription = customIntervalObservable.subscribe(data => {
      console.log(data);
    }, error => {
      console.log(error);
      alert(error.message);
    }, () => {
      console.log('Completed');
    });
  }
```
**Note**: `Observable.create()` has been **deprecated**. Instead, use:

```typescript
this.observableSubscription= new Observable((observer: Observer) => {
  observer.next();
  observer.complete();
});
```

### Understanding Operators:
- See [Rxjs operators](https://rxjs.dev/guide/operators)
```typescript
import {Component, OnDestroy, OnInit} from '@angular/core';

import {interval, Observable, Subscription} from "rxjs";
import {filter, map} from "rxjs/operators";

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit, OnDestroy {

  private firstObsSubscription: Subscription;

  constructor() { }

  ngOnInit() {
    const customIntervalObservable = Observable.create(observer => {
      let count = 0;
      setInterval(() => {
        observer.next(count);
        if(count === 2){
          observer.complete();
        }
        if(count > 3) {
          observer.error(new Error('Count is greater 3!!!'))
        }
        count++;
      }, 1000)
    });

    this.firstObsSubscription = customIntervalObservable.pipe(filter(data => {
      return data > 0;
    }), map((data: number) => {
      return 'Round: ' + (data + 1);
    })).subscribe(data => {
      console.log(data);
    }, error => {
      console.log(error);
      alert(error.message);
    }, () => {
      console.log('Completed');
    });
  }

  ngOnDestroy() {
    this.firstObsSubscription.unsubscribe();
  }
}
```

### Subjects
**(Note: Use subjects over event emitters EXCEPT when using `@Output`)**\
Create subject in service:
```typescript
import {EventEmitter, Injectable} from "@angular/core";
import { Subject } from 'rxjs';

@Injectable({providedIn: "root"})
export class UserService {
  // activatedEmitter = new EventEmitter<boolean>();
  activatedEmitter = new Subject<boolean>();
}
```

Call `next()`:
```typescript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Params } from '@angular/router';
import { UserService } from "../home/user.service";

@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrls: ['./user.component.css']
})
export class UserComponent implements OnInit {
  id: number;

  constructor(private route: ActivatedRoute, private userService: UserService) {}

  ngOnInit() {
    this.route.params.subscribe((params: Params) => {
      this.id = +params.id;
    });
  }

  onActivate() {
    this.userService.activatedEmitter.next(true);
  }
}
```
Call `onActivate()` in `user.component.html`:
```angular2html
<p>User with <strong>ID {{ id }}</strong> was loaded</p>
<button class="btn btn-primary" (click)="onActivate()">Activate</button>
```

Subscribe to subject in `app.component.ts`:

```typescript
import { Component, OnInit } from '@angular/core';
import {UserService} from "./home/user.service";

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  userActivated = false;
  constructor(private userService: UserService) {}

  ngOnInit() {
    this.userService.activatedEmitter.subscribe(didActivate => {
      this.userActivated = didActivate;
    });
  }
}
```

Use the flag in `app.component.html`:
```angular2html
  <p *ngIf="userActivated">Activated</p>
```

## Handling Forms in Angular Apps

### Two approaches
1. **Template-Driven** (Angular infers the Form Object from the DOM)
2. **Reactive** (Form is created programmatically and synchronized with the DOM)

### Template driven approach

```typescript
import {Component, ViewChild} from '@angular/core';
import {NgForm} from "@angular/forms";

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  @ViewChild('f') signupForm: NgForm;
  defaultQuestion = 'pet';
  answer: string;
  genders = ['male', 'female'];
  
  suggestUserName() {
    const suggestedName = 'Superuser';
  }
  
  onSubmit() {
    console.log(this.signupForm);
  }
}
```

```angular2html
<div class="container">
  <div class="row">
    <div class="col-xs-12 col-sm-10 col-md-8 col-sm-offset-1 col-md-offset-2">
      <form (ngSubmit)="onSubmit()" #f="ngForm">
        <div
          id="user-data"
          ngModelGroup="userData"
          #userData="ngModelGroup"
        >
          <div class="form-group">
            <label for="username">Username</label>
            <input
              type="text"
              id="username"
              class="form-control"
              ngModel
              name="username"
              required
            >
          </div>
          <button class="btn btn-default" type="button">Suggest an Username</button>
          <div class="form-group">
            <label for="email">Mail</label>
            <input
              type="email"
              id="email"
              class="form-control"
              ngModel
              name="email"
              required
              email
              #email="ngModel"
            >
        <span class="help-block" *ngIf="!email.valid && email.touched">Please enter a valid value!</span>
          </div>
        </div>
        <p *ngIf="!userData.valid && userData.touched">User Data is invalid!</p>
        <div class="form-group">
          <label for="secret">Secret Questions</label>
          <select
            id="secret"
            class="form-control"
            [ngModel]="defaultQuestion"
            name="secret"
          >
            <option value="pet">Your first Pet?</option>
            <option value="teacher">Your first teacher?</option>
          </select>
        </div>
        <div class="form-group">
          <textarea
            name="questionAnswer"
            rows="3"
            [(ngModel)]="answer"
          >
          </textarea>
        </div>
        <p>Your reply: {{answer}}</p>
        <div class="radio" *ngFor="let gender of genders">
          <label>
            <input
              type="radio"
              name="gender"
              ngModel
              [value]="gender"
              required
            >
            {{gender}}
          </label>
        </div>
        <button
          class="btn btn-primary"
          type="submit"
          [disabled]="!f.valid"
        >Submit</button>
      </form>
    </div>
  </div>
</div>

```

#### Setting and Patching Form Values
Set: (for entire form)
```typescript
  suggestUserName() {
    const suggestedName = 'Superuser';
    this.signupForm.setValue({
      userData: {
        username: suggestedName,
        email: ''
      },
      secret: 'pet',
      questionAnswer: '',
      gender: 'male'
    })
  }
```

Patch: (for single item)
```typescript
  suggestUserName() {
    const suggestedName = 'Superuser';
    this.signupForm.form.patchValue({
      userData: {
        username: suggestedName
      }
    });
  }
```

### Reactive Approach
(Note: need to import ReactiveFormsModule in `app.module.ts`)
```typescript
import {Component, OnInit} from '@angular/core';
import {FormArray, FormControl, FormGroup, Validators} from "@angular/forms";
import {Observable} from "rxjs";

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  genders = ['male', 'female'];
  signupForm: FormGroup;
  forbiddenUsernames = ['Chris', 'Anna'];

  ngOnInit() {
    this.signupForm = new FormGroup({
      'userData': new FormGroup({
      'username': new FormControl(null, [Validators.required, this.forbiddenNames.bind(this)]),
      'email': new FormControl(null, [Validators.required, Validators.email], this.forbiddenEmails)
      }),
      'gender': new FormControl('male'),
      'hobbies': new FormArray([])
    });
    // this.signupForm.valueChanges.subscribe(
    //   (value) => console.log(value)
    // );
    this.signupForm.statusChanges.subscribe(
      (status) => console.log(status)
    );
    // this.signupForm.setValue({
    //   'userData': {
    //     'username': 'Max',
    //     'email': 'max@test.com'
    //   },
    //   'gender': 'male',
    //   'hobbies': []
    // })
    this.signupForm.patchValue({
      'userData': {
        'username': 'Max',
      }
    })
  }

  onSubmit() {
    console.log(this.signupForm)
    this.signupForm.reset();
  }

  getControls() {
    return (<FormArray>this.signupForm.get('hobbies')).controls;
  }

  onAddHobby() {
    const control = new FormControl(null, Validators.required);
    (<FormArray>this.signupForm.get('hobbies')).push(control);
  }

  // Custom validator
  forbiddenNames(control: FormControl): {[s: string]: boolean} {
    if (this.forbiddenUsernames.indexOf(control.value) !== -1) {
      return {'nameIsForbidden': true};
    }
    return null;
  }

  // Async validator
  forbiddenEmails(control: FormControl): Promise<any> | Observable<any> {
    const promise = new Promise<any>((resolve, reject) => {
      setTimeout(() => {
        if(control.value  === 'test@test.com') {
          resolve({'emailIsForbidden': true });
        } else {
          resolve(null);
        }
      },1500)
    });
    return promise;
  }
}

```

```angular2html
<div class="container">
  <div class="row">
    <div class="col-xs-12 col-sm-10 col-md-8 col-sm-offset-1 col-md-offset-2">
      <form [formGroup]="signupForm" (ngSubmit)="onSubmit()" >
        <div formGroupName="userData">
          <div class="form-group">
            <label for="username">Username</label>
            <input
              type="text"
              id="username"
              formControlName="username"
              class="form-control">
            <span
              *ngIf="!signupForm.get('userData.username').valid && signupForm.get('userData.username').touched"
              class="help-block">
              <span *ngIf="signupForm.get('userData.username').errors['nameIsForbidden']">This name is invalid!</span>
              <span *ngIf="signupForm.get('userData.username').errors['required']">This field is required!</span>
              </span>
          </div>
          <div class="form-group">
            <label for="email">email</label>
            <input
              type="text"
              id="email"
              formControlName="email"
              class="form-control">
            <span
              *ngIf="!signupForm.get('userData.email').valid && signupForm.get('userData.email').touched"
              class="help-block">Please enter a valid email!</span>
          </div>
        </div>
        <div class="radio" *ngFor="let gender of genders">
          <label>
            <input
              type="radio"
              formControlName="gender"
              [value]="gender">{{ gender }}
          </label>
        </div>
        <div formArrayName="hobbies">
          <h4>Your hobbies:</h4>
          <button
            class="btn btn-default"
            type="button"
            (click)="onAddHobby()">Add Hobby</button>
          <div
            class="form-group"
            *ngFor="let hobbyControl of getControls(); let i = index"
          >
            <input type="text" class="form-control" [formControlName]="i">
          </div>
        </div>
        <button
          class="btn btn-primary"
          type="submit"
          [disabled]="!signupForm.valid"
        >
          Submit
        </button>
        <span
          *ngIf="!signupForm.valid && signupForm.touched"
          class="help-block">Please enter a valid data!</span>
      </form>
    </div>
  </div>
</div>
```

## Pipes

### Creating a custom pipe

```typescript
import {Pipe, PipeTransform} from "@angular/core";

@Pipe({
  name: 'shorten'
})
export class ShortenPipe implements PipeTransform {

  transform(value: any, limit: number) {
    if(value.length > limit) {
      return value.substr(0, limit) + ' ...';
    }
    return value;
  }
}
```

```angular2html
<div class="container">
  <div class="row">
    <div class="col-xs-12 col-sm-10 col-md-8 col-sm-offset-1 col-md-offset-2">
      <ul class="list-group">
        <li
          class="list-group-item"
          *ngFor="let server of servers"
          [ngClass]="getStatusClasses(server)">
          <span
            class="badge">
            {{ server.status }}
          </span>
          <strong>{{ server.name | shorten: 15}}</strong> |
          {{ server.instanceType | uppercase }} |
          {{ server.started | date:'fullDate' | uppercase }}
        </li>
      </ul>
    </div>
  </div>
</div>
```
## Making HTTP Requests

```typescript
import {Component, OnDestroy, OnInit} from '@angular/core';
import { HttpClient } from '@angular/common/http';
import {map} from "rxjs/operators";
import {Post} from "./post.model";
import {PostService} from "./post.service";
import {Subscription} from "rxjs";

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit, OnDestroy {
  // Note that we manage the state from the component
  loadedPosts: Post[] = [];
  isFetching = false;
  error = null;
  private errorSubscription: Subscription;

  constructor(private http: HttpClient, private postsService: PostService) {}

  ngOnInit() {
    // Using subjects for error handling
    this.errorSubscription = this.postsService.error.subscribe(errorMessage => {
      this.error = errorMessage;
    });

    this.isFetching = true;
    this.postsService.fetchPosts().subscribe(posts =>{
      this.isFetching = false;
      this.loadedPosts = posts;
    }, error => {
      this.error = error.message;
    });
  }

  onCreatePost(postData: Post) {
    // Send Http request using service implementation
    this.postsService.createAndStorePost(postData.title, postData.content);
  }

  onFetchPosts() {
    // Send Http request
    this.isFetching = true;
    this.postsService.fetchPosts().subscribe(posts => {
      this.isFetching = false;
      this.loadedPosts = posts;
    }, error => {
      this.error = error.message;
    });
  }

  onClearPosts() {
    // Send Http request
    this.postsService.deletePosts().subscribe(() => {
      this.loadedPosts = [];
    });
  }

  ngOnDestroy() {
    this.errorSubscription.unsubscribe();
  }

  onHandleError() {
    this.error = null;
  }
}

```

Here is the service that makes the http request.\
(we don't subscribe to the observable here, instead we can do it in the component [see above])
```typescript
import {Injectable} from "@angular/core";
import {HttpClient} from "@angular/common/http";
import {Post} from "./post.model";
import {catchError, map} from "rxjs/operators";
import {Subject, throwError} from "rxjs";

@Injectable({providedIn: 'root'})
export class PostService {
  baseUrl = 'https://ng-complete-guide-56fe0-default-rtdb.firebaseio.com/posts.json';
  // An alternative way of error handling
  error = new Subject<string>();

  constructor(private http: HttpClient) {}

  createAndStorePost(title: string, content: string) {
    const postData: Post = {title: title, content: content};
    this.http
      .post<{name: string}>(
        this.baseUrl,
        postData
      ).subscribe(responseData => {
      console.log(responseData);
    }, error => {
      this.error.next(error.message);
    });
  }
    
  // Notice that by returning the http request we can subscribe in the component
   fetchPosts() {
    return this.http
      .get<{ [key: string]: Post}>(this.baseUrl)
      .pipe(
        map((responseData) => {
          const postsArray: Post[] = [];
          for (const key in responseData) {
            if (responseData.hasOwnProperty(key)) {
              postsArray.push({ ...responseData[key], id: key})
            }
          }
          return postsArray;
        }),
        catchError(errorResponse => {
          // Send to analytics server
          return throwError(errorResponse);
        })
      );
  }

  deletePosts() {
    return this.http.delete(this.baseUrl)
  }
}
```

The html looks like:
```angular2html
<div class="container">
  <div class="row">
    <div class="col-xs-12 col-md-6 col-md-offset-3">
      <form #postForm="ngForm" (ngSubmit)="onCreatePost(postForm.value)">
        <div class="form-group">
          <label for="title">Title</label>
          <input
            type="text"
            class="form-control"
            id="title"
            required
            ngModel
            name="title"
          />
        </div>
        <div class="form-group">
          <label for="content">Content</label>
          <textarea
            class="form-control"
            id="content"
            required
            ngModel
            name="content"
          ></textarea>
        </div>
        <button
          class="btn btn-primary"
          type="submit"
          [disabled]="!postForm.valid"
        >
          Send Post
        </button>
      </form>
    </div>
  </div>
  <hr />
  <div class="row">
    <div class="col-xs-12 col-md-6 col-md-offset-3">
      <button class="btn btn-primary" (click)="onFetchPosts()">
        Fetch Posts
      </button>
      |
      <button
        class="btn btn-danger"
        [disabled]="loadedPosts.length < 1"
        (click)="onClearPosts()"
      >
        Clear Posts
      </button>
    </div>
  </div>
  <div class="row">
    <div class="col-xs-12 col-md-6 col-md-offset-3">
      <p *ngIf="loadedPosts.length < 1 && !isFetching">No posts available!</p>
      <ul class="list-group" *ngIf="loadedPosts.length >= 1 && !isFetching">
        <li class="list-group-item" *ngFor="let post of loadedPosts">
          <h1>{{ post.title }}</h1>
          <p>{{ post.content }}</p>
        </li>
      </ul>
      <p *ngIf="isFetching && !error">Loading...</p>
      <div class="alert alert-danger" *ngIf="error">
        <h1>An error occurred!</h1>
        <p>{{error}}</p>
        <button class="btn btn-danger" (click)="onHandleError()">Okay</button>
      </div>
    </div>
  </div>
</div>

```

### Setting Headers

```typescript
this.http
  .get<{ [key: string]: Post}>(
    this.baseUrl,
    {
      headers: new HttpHeaders({ 'Custom-Header': 'Hello' })
    }
  )
```
Inspecting the request we see:
```javascript
Custom-Header: Hello
```

### Adding query parameters
