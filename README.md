## A beginest implementation of MediatR from .NET C# to Angular

The mediator pattern is one of the ways to arrange a loosely coupled communication between components. I personally think that's a very interesting way.

This implementation looks to [Jimmy Bogard's](https://github.com/jbogard) idea when he did implement mediator pattern in .net c# on [MediatR](https://github.com/jbogard/MediatR) component disponible in [nuget.org](https://www.nuget.org/packages/MediatR).


Thank's Jimmy for you contribution.

Now, let's rock.

### Interfaces and Mediator Service

#### IRequest
Interface to write commands, queries or notifications
```typescript
export interface IRequest<TResponse> { } 
```

#### IRequest Handler
Interface to write handlers
```typescript
import { IRequest } from "./irequest.interface";

export interface IRequestHandler<RequestType extends IRequest<TResponse>, TResponse>
{
    Handle(request: RequestType): TResponse;
}
```

#### Mediator Service
Service to get command based handler and execute it.
```typescript
import { Injectable, Injector } from '@angular/core';
import { IRequestHandler } from './irequest-handler.interface';
import { IRequest } from './irequest.interface';

@Injectable({
    providedIn: 'root',
})
export class MediatorService {

    constructor(private inject: Injector) { }

    send(request: IRequest<Response>): Response 
    {
        let process = <IRequestHandler<IRequest<Response>, Response>>this.inject.get(request.constructor);
        let response = process.Handle(request);
        return response;
    }
}
```
&nbsp;
&nbsp;
### Commands, Queries and Handlers
#### Writing It
*get-profile-by-id.query*
```typescript
import { IRequest } from "src/app/shared/pattern/mediator";
import { ProfileModel } from "../models/profile.model";

export class GetProfileByIdQuery implements IRequest<ProfileModel> {
    id: number;

    constructor(id: number){
        this.id = id;
    }
}
```
&nbsp;
*get-profile-by-id.query-handler*
```typescript
import { HttpClient } from '@angular/common/http';
import { IRequestHandler } from "src/app/shared/pattern/mediator";
import { ProfileModel } from "../models/profile.model";
import { GetProfileByIdQuery } from "./get-profile-by-id.query";

export class GetProfileByIdQueryHandler implements IRequestHandler<GetProfileByIdQuery, ProfileModel>{

    Handle(request: GetProfileByIdQuery): ProfileModel {

        let response = new ProfileModel();
        response.id = request.id;
        response.name = 'Paulo ' + request.id.toString();
        return response;

    }    
}
```
&nbsp;
&nbsp;
### Gluing all
#### Configure Module

You need to create a **mediator module** or use **some module** to configure the service.

```typescript
import { HttpClient } from '@angular/common/http';
import { Injector, NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { GetProfileByIdQuery } from './profile/service/get-profile-by-id.query';
import { GetProfileByIdQueryHandler } from './profile/service/get-profile-by-id.query-handler';
import { MediatorService } from 'src/app/shared/pattern/mediator';

@NgModule({
  declarations: [],
  imports: [
    CommonModule
  ],
  providers: [
    //Mediator service added to use injcted in some component.
    { provide: MediatorService, useClass: MediatorService, deps: [ Injector ]},
  
    //Queries
    { provide: GetProfileByIdQuery, useClass: GetProfileByIdQueryHandler }

    //Commands

    //Notifications
    
  ]   
  
})
export class MediatorModule { }
```
&nbsp;
&nbsp;
### To use
Inject mediator service on a target component and **send** a query, command or notification.

So that's it, simple as that.
