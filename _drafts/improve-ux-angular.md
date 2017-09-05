---
title: Improve your general User Experience when working with Angular or Ionic
layout: post
author: corentin
cover: 
tags:
- ux
- front-end
---

There are a few simple tricks to improve the global user experience in your Angular/Ionic apps that might be unknown.

Let's shed some light on them!

---

# Summary
{:.no_toc}

* TOC
{:toc}

---

# Cache some data locally

One common thing web apps do is use data. But what happens when those HTTP requests fail or take longer than expected?

Well, the most common solution is to use some cache! But, how do you do that without breaking most of your codebase?

Simple! Your just change your service to handle all that for you while making the most out of `Observable`s.

## Before

Here is what a basic HTTP request looks like in an Angular service (looks familiar?):

```
import { Http, Response } from '@angular/http';
import { Observable } from 'rxjs/Observable';
import { SuperHero } from '../models/super-hero';
import { API_ROOT } from '../my-config';

@Injectable()
export class MySuperService {
	constructor(private http: Http) {}
	
	public getSuperHeroes(): Observable<SuperHero[]> {
		return this.http.get(`${API_ROOT}/super-heroes`)
			.map((res: Response) => res.json() as SuperHero[]);
	}
}
```

Now, let's turn that into a more powerful tool!

```
import { Http, Response } from '@angular/http';
import { Observable } from 'rxjs/Observable';
import { SuperHero } from '../models/super-hero';
import { MyStorage } from './my-storage';
import { API_ROOT } from '../my-config';

@Injectable()
export class MySuperService {
	constructor(
		private http: Http,
		private storage: MyStorage
	) {}
	
	public getSuperHeroes(): Observable<SuperHero[]> {
		return new Observable((observer) => {
			this.storage.get('super-heroes')
			.then((data: SuperHero[]) => {
				if (data) {
					observer.next(data);
				}
				
				this.http.get(`${API_ROOT}/super-heroes`)
				.toPromise() // You can also use an Observable, but for demonstration sake I'll use a regular Promise
				.then((res: Response) => res.json())
				.then((data: SuperHero[]) => {
					observer.next(data);
					observer.complete();
				})
				.catch((err) => {
					observer.error(err);
				});
			});
		});
	}
}
```

So, what does this code do?
