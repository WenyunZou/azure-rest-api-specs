# Runner Behavior

This document explains the expected behavior of runner.

## Load Test Scenario via OAV

You could load the test scenario file via oav. It would be resolved as a simple object.

```typescript
  const readmeMd: string =
    "/home/username/azure-rest-api-specs/specification/containerservice/resource-manager/readme.md";
  const argv = {
    ["try-require"]: "readme.test.md",
    tag: "package-2020-12",
  };

  // Get input-file config in readme.md
  const autorestConfig = await getAutorestConfig(argv, readmeMd);
  const swaggerFilePaths: string[] = autorestConfig["input-file"];
  const fileRoot = dirname(readmeMd);

  console.log("input-file:");
  console.log(swaggerFilePaths);

  // Create the loader from OAV
  const loader = TestResourceLoader.create({
    useJsonParser: false,
    checkUnderFileRoot: false,
    fileRoot,
    swaggerFilePaths,
  });

  // Load the test scenario file. File list could also be specified in readme.test.md
  const testDef = await loader.load(
    "Microsoft.ContainerService/stable/2020-12-01/test-scenarios/containerService.yaml"
  );

  console.log(testDef.testScenarios[0].steps);

  // Setup initial variable env
  const env = new VariableEnv();
  env.setBatch({
    subscriptionId: "db5eb68e-73e2-4fa8-b18a-46cd1be4cce5",
    location: "westus",
    SSH_PUBLIC_KEY: "__public_key_ssh__",
  });

  // Reference runner implementation in OAV. You need to implement your own runner.
  const runner = new TestScenarioRunner({
    jsonLoader: loader.jsonLoader,
    env,
    client: new TestScenarioRestClient(getDefaultAzureCredential(), {}),
  });

  try {
    for (const scenario of testDef.testScenarios) {
      await runner.executeScenario(scenario);
    }
  } catch (e) {
    console.log(e.message, e.stack);
  } finally {
    console.timeLog("TestLoad");
    await runner.cleanAllTestScope();
  }
```

After the test scenario is loaded, the test step will be slightly different from the file content. Every step will have the following fields:

- requestParameters
  - 