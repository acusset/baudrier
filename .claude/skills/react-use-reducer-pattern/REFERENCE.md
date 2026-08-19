# Example

Before — 3 related states, updated together in a few places:

```tsx
function AudioPlayer({ tracks }: { tracks: Track[] }) {
  const [trackIndex, setTrackIndex] = useState(0);
  const [isPlaying, setIsPlaying] = useState(false);
  const [volume, setVolume] = useState(1);

  function handleTrackEnd() {
    if (trackIndex < tracks.length - 1) {
      setTrackIndex((i) => i + 1);
      setIsPlaying(true);
    } else {
      setIsPlaying(false);
      setTrackIndex(0);
    }
  }

  function handleMute() {
    setVolume(0);
    setIsPlaying(false);
  }
  // ...
}
```

After — one reducer, transitions computed atomically:

```tsx
type State = { trackIndex: number; isPlaying: boolean; volume: number };
type Action =
  | { type: 'trackEnded'; trackCount: number }
  | { type: 'muted' }
  | { type: 'volumeChanged'; volume: number };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'trackEnded':
      return action.trackCount - 1 > state.trackIndex
        ? { ...state, trackIndex: state.trackIndex + 1, isPlaying: true }
        : { ...state, trackIndex: 0, isPlaying: false };
    case 'muted':
      return { ...state, volume: 0, isPlaying: false };
    case 'volumeChanged':
      return { ...state, volume: action.volume };
  }
}

function AudioPlayer({ tracks }: { tracks: Track[] }) {
  const [state, dispatch] = useReducer(reducer, { trackIndex: 0, isPlaying: false, volume: 1 });

  function handleTrackEnd() {
    dispatch({ type: 'trackEnded', trackCount: tracks.length });
  }

  function handleMute() {
    dispatch({ type: 'muted' });
  }
  // ...
}
```

Each event handler now issues a single `dispatch` instead of a sequence of setter calls that have to stay in sync by hand. The `trackEnded` case bundles `trackIndex` and `isPlaying` into one atomic transition, and `muted` bundles `volume` and `isPlaying`.
