# [openapi-generator](https://github.com/OpenAPITools/openapi-generator) _[typescript-fetch](https://openapi-generator.tech/docs/generators/typescript-fetch)_
## Middleware
```typescript
/**
 * Small middleware to add appropriate headers depending on the HTTP method
 * before starting the API call.
 */
export class AppropriateOptionsMiddleware implements Middleware {
    private getCookie(name: string) {
        let cookieValue = null;
        if (document.cookie && document.cookie !== "") {
            const cookies = document.cookie.split(";");
            for (let i = 0; i < cookies.length; i++) {
                const cookie = cookies[i].trim();
                // Does this cookie string begin with the name we want?
                if (cookie.substring(0, name.length + 1) === name + "=") {
                    cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                    break;
                }
            }
        }
        return cookieValue;
    }

    /**
     * Called before executing the request.
     * @param context
     */
    pre(context: RequestContext): Promise<FetchParams | void> {
        const init = context.init;

        const getHeaders = (): HeadersInit =>
            ["POST"].includes(init.method || "")
                ? { "X-CSRFToken": this.getCookie("csrftoken") || "", "Content-Type": "application/json" }
                : {};

        context.init = { ...init, headers: getHeaders() };
        return Promise.resolve({ url: context.url, init: context.init });
    }
}

DefaultConfig.config = new Configuration({
    credentials: "include",
    middleware: [new AppropriateOptionsMiddleware()],
});
```

## [drf-spectacular](https://github.com/tfranzel/drf-spectacular)
```python
# file settings.py
SPECTACULAR_SETTINGS = {
    ...
    "POSTPROCESSING_HOOKS": [
        "backend.spectacular_hooks.mark_all_outputs_required",
        "drf_spectacular.hooks.postprocess_schema_enums",
        "backend.spectacular_hooks.add_x_enum_varnames",
    ],
}


# file spectacular_hooks.py
import re
from typing import Any


def mark_all_outputs_required(result: Any, **kwargs: Any):
    schemas = result.get("components", {}).get("schemas", {})
    for name, schema in schemas.items():
        if name.endswith("Request"):
            continue
        schema["required"] = sorted(schema["properties"].keys())
    return result


def add_x_enum_varnames(result: Any, **kwargs: Any):
    """
    Adds x-enum-varnames based on drf-spectacular names
    :param result:
    :param kwargs:
    :return:
    """
    schemas = result.get("components", {}).get("schemas", {})
    for name, schema in schemas.items():
        if name.endswith("Enum"):
            description: str = schema.get("description")
            if description:
                varnames = []
                descriptions = []
                for line in description.splitlines():
                    name_match = re.search(r" - (.+)$", line)
                    if name_match:
                        descriptions.append(name_match.group(1))
                        varnames.append(
                            name_match.group(1)
                            .replace(" ", "_")
                            .replace("ä", "ae")
                            .replace("ö", "oe")
                            .replace("ü", "ue")
                            .capitalize()
                        )
                schema["x-enum-varnames"] = varnames
                schema["x-enum-descriptions"] = descriptions

    return result

```
