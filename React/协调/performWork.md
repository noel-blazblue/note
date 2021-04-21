### performSyncWorkOnRoot

从`root`开始同步执行工作

1. `flushPassiveEffects`刷新副作用，调度如`useEffect`之类的回调
2. 从`root`节点执行`render`，`renderRootSync`
3. `render`执行完毕，链接`root.finishedWork`为`wip fiber`，然后执行`commitRoot`
4. 确保是否有任务还在等待调度，`ensureRootIsScheduled`

```js
function performSyncWorkOnRoot(root) {
  if (enableProfilerTimer && enableProfilerNestedUpdatePhase) {
    syncNestedUpdateFlag();
  }

  invariant(
    (executionContext & (RenderContext | CommitContext)) === NoContext,
    'Should not already be working.',
  );

  // 刷新副作用，如useEffect
  flushPassiveEffects();

  let lanes;
  let exitStatus;
  if (
    root === workInProgressRoot &&
    includesSomeLane(root.expiredLanes, workInProgressRootRenderLanes)
  ) {
    // There's a partial tree, and at least one of its lanes has expired. Finish
    // rendering it before rendering the rest of the expired work.
    lanes = workInProgressRootRenderLanes;
    exitStatus = renderRootSync(root, lanes);
    if (
      includesSomeLane(
        workInProgressRootIncludedLanes,
        workInProgressRootUpdatedLanes,
      )
    ) {
      // The render included lanes that were updated during the render phase.
      // For example, when unhiding a hidden tree, we include all the lanes
      // that were previously skipped when the tree was hidden. That set of
      // lanes is a superset of the lanes we started rendering with.
      //
      // Note that this only happens when part of the tree is rendered
      // concurrently. If the whole tree is rendered synchronously, then there
      // are no interleaved events.
      lanes = getNextLanes(root, lanes);
      exitStatus = renderRootSync(root, lanes);
    }
  } else {
    lanes = getNextLanes(root, NoLanes);
    exitStatus = renderRootSync(root, lanes);
  }

  if (root.tag !== LegacyRoot && exitStatus === RootErrored) {
    executionContext |= RetryAfterError;

    // If an error occurred during hydration,
    // discard server response and fall back to client side render.
    if (root.hydrate) {
      root.hydrate = false;
      clearContainer(root.containerInfo);
    }

    // If something threw an error, try rendering one more time. We'll render
    // synchronously to block concurrent data mutations, and we'll includes
    // all pending updates are included. If it still fails after the second
    // attempt, we'll give up and commit the resulting tree.
    lanes = getLanesToRetrySynchronouslyOnError(root);
    if (lanes !== NoLanes) {
      exitStatus = renderRootSync(root, lanes);
    }
  }

  if (exitStatus === RootFatalErrored) {
    const fatalError = workInProgressRootFatalError;
    prepareFreshStack(root, NoLanes);
    markRootSuspended(root, lanes);
    ensureRootIsScheduled(root, now());
    throw fatalError;
  }

  // We now have a consistent tree. Because this is a sync render, we
  // will commit it even if something suspended.
  const finishedWork: Fiber = (root.current.alternate: any);
  root.finishedWork = finishedWork;
  root.finishedLanes = lanes;
  commitRoot(root);

  // Before exiting, make sure there's a callback scheduled for the next
  // pending level.
  // 在退出之前，确保是否还有任务在等待调度
  ensureRootIsScheduled(root, now());

  return null;
}
```



### renderRootSync

执行`workLoopSync`

```js
function renderRootSync(root: FiberRoot, lanes: Lanes) {
  const prevExecutionContext = executionContext;
  executionContext |= RenderContext;
  const prevDispatcher = pushDispatcher();

  // If the root or lanes have changed, throw out the existing stack
  // and prepare a fresh one. Otherwise we'll continue where we left off.
  if (workInProgressRoot !== root || workInProgressRootRenderLanes !== lanes) {
    prepareFreshStack(root, lanes);
    startWorkOnPendingInteractions(root, lanes);
  }

  const prevInteractions = pushInteractions(root);

  if (enableSchedulingProfiler) {
    markRenderStarted(lanes);
  }

  do {
    try {
      workLoopSync();
      break;
    } catch (thrownValue) {
      handleError(root, thrownValue);
    }
  } while (true);
  resetContextDependencies();
  if (enableSchedulerTracing) {
    popInteractions(((prevInteractions: any): Set<Interaction>));
  }

  executionContext = prevExecutionContext;
  popDispatcher(prevDispatcher);

  if (workInProgress !== null) {
    // This is a sync render, so we should have finished the whole tree.
    invariant(
      false,
      'Cannot commit an incomplete root. This error is likely caused by a ' +
        'bug in React. Please file an issue.',
    );
  }
  if (enableSchedulingProfiler) {
    markRenderStopped();
  }

  // Set this to null to indicate there's no in-progress render.
  workInProgressRoot = null;
  workInProgressRootRenderLanes = NoLanes;

  return workInProgressRootExitStatus;
}
```



### renderRootConcurrent

```js
function renderRootConcurrent(root: FiberRoot, lanes: Lanes) {
  const prevExecutionContext = executionContext;
  executionContext |= RenderContext;
  const prevDispatcher = pushDispatcher();

  // If the root or lanes have changed, throw out the existing stack
  // and prepare a fresh one. Otherwise we'll continue where we left off.
  if (workInProgressRoot !== root || workInProgressRootRenderLanes !== lanes) {
    resetRenderTimer();
    prepareFreshStack(root, lanes);
    startWorkOnPendingInteractions(root, lanes);
  }

  const prevInteractions = pushInteractions(root);

  if (enableSchedulingProfiler) {
    markRenderStarted(lanes);
  }

  do {
    try {
      workLoopConcurrent();
      break;
    } catch (thrownValue) {
      handleError(root, thrownValue);
    }
  } while (true);
  resetContextDependencies();
  if (enableSchedulerTracing) {
    popInteractions(((prevInteractions: any): Set<Interaction>));
  }

  popDispatcher(prevDispatcher);
  executionContext = prevExecutionContext;

  // Check if the tree has completed.
  if (workInProgress !== null) {
    // Still work remaining.
    if (enableSchedulingProfiler) {
      markRenderYielded();
    }
    return RootIncomplete;
  } else {
    // Completed the tree.
    if (enableSchedulingProfiler) {
      markRenderStopped();
    }

    // Set this to null to indicate there's no in-progress render.
    workInProgressRoot = null;
    workInProgressRootRenderLanes = NoLanes;

    // Return the final exit status.
    return workInProgressRootExitStatus;
  }
}
```



### workLoopSync

深度优先遍历`fiber`。

```js
function workLoopSync() {
  // Already timed out, so perform work without checking if we need to yield.
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}
```



### workLoopConcurrent

循环执行每个`fiber`，并在每一次执行完了之后，检查是否还有剩余的任务时间，如果没有则跳出

```js
function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```



### performUnitOfWork

- 执行每个`beginWork`去处理`fiber`，它会返回经过`reconcile`的`child`，然后赋值`child`给`workInProgress`，这样在上层的`workLoopSync`就可以继续遍历。实现一个深度优先遍历
- 如果处理完之后`child`不存在，说明已经到了叶子节点，那就执行`completeUnitOfWork`。
  - `completeUnitOfWork`判断当前节点存在`sibling`的情况下会将其赋值给`workInProgress`，这样会继续``workLoopSync``的遍历

```js
function performUnitOfWork(unitOfWork: Fiber): void {
  // The current, flushed, state of this fiber is the alternate. Ideally
  // nothing should rely on this, but relying on it here means that we don't
  // need an additional field on the work in progress.
  const current = unitOfWork.alternate;

  let next;
  // beginWork会返回经过了reconcile的child
  if (enableProfilerTimer && (unitOfWork.mode & ProfileMode) !== NoMode) {
    startProfilerTimer(unitOfWork);
    next = beginWork(current, unitOfWork, subtreeRenderLanes);
    stopProfilerTimerIfRunningAndRecordDelta(unitOfWork, true);
  } else {
    next = beginWork(current, unitOfWork, subtreeRenderLanes);
  }

  // render执行完毕，更新当前fiber的props
  unitOfWork.memoizedProps = unitOfWork.pendingProps;
  if (next === null) {
    // If this doesn't spawn new work, complete the current work.
    // 遍历到了叶子节点，可以完成工作。
    completeUnitOfWork(unitOfWork);
  } else {
    workInProgress = next;
  }

  ReactCurrentOwner.current = null;
}
```

