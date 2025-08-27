# 简单内存池

```c
#define CHUNK_SIZE 1000

struct chunk {
    struct chunk * next;
};

struct pool {
    struct chunk * header;
    struct chunk * current;
    int current_used;
};

static void pool_init(struct pool *p) {
    p->header = NULL;
    p->current = NULL;
    p->current_used = 0;
}

static void pool_release(struct pool *p) {
    struct chunk *tmp = p->header;
    while (tmp) {
        struct chunk *p = tmp->next;
        free(tmp);
        tmp = p;
    }
}

static void *
pool_newchunk(struct pool *p, size_t sz) {
    struct chunk * t = malloc(sizeof(struct chunk) + sz);
    if (t == NULL)
        return NULL;
    t->next = p->header;
    p->header = t;
    return t+1;
}

static void *
pool_alloc(struct pool *p, size_t sz) {
    // align by 8
    sz = (sz + 7) & ~7;
    if (sz >= CHUNK_SIZE) {
        return pool_newchunk(p, sz);
    }
    if (p->current == NULL) {
        if (pool_newchunk(p, CHUNK) == NULL) {
            return NULL;
        }
        p->current = p->header;
    }
    
    if (sz + p->current_used <= CHUNK_SIZE) {
        void * ret = (char *)(p->current+1) + p->current_used;
        p->current_used += sz;
        return ret;
    }

    if (sz >= p->current_used) {
        return pool_newchunk(pool, sz);
    } else {
        void * ret = pool_newchunk(pool, CHUNK_SIZE);
        p->current = p->header;
        p->current_used = sz;
        return ret;
    }
}
```